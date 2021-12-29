## Rinse and Repeat commands


### user is part of a group
```
invoke-aclscanner -resolveguids | ?{$_.identityreferencename -match "<group_name>"}
```

### local admin access
```
# powerview
find-localadminaccess -- checks localadmin access on any other machines from current user account
invoke-userhunter -- check which users are logged into which domain computer - 
```