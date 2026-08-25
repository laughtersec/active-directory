---
tags:
  - domain-enumeration
---
[[PowerShell]] on steroids: 

[Documentation](https://powersploit.readthedocs.io/en/latest/)

```powershell title:"Importing"
. .\PowerView.ps1
Import-Module .\PowerView.ps1
```

```powershell title:"Get the current domain"
Get-Domain
```

```powershell title:"Get object of another domain"
Get-Domain -Domain domain.name
```

```powershell title:"Get domain SID for the current domain"
Get-DomainSID
```

```powershell title:"Get domain policy for the current domain"
1. Get-DomainPolicyData
2. (Get-DomainPolicyData).systemaccess
```

```powershell title:"Get domain policy for another domain"
(Get-DomainPolicyData -domain domain.name).systemaccess
```

###### Finding the Domain Controller

```powershell title:"Get the domain controller for the current domain"
Get-DomainController
```

```powershell title:"Get the domain controller for another domain"
Get-DomainController -Domain domain.com
```

###### Finding users in a domain

```powershell title:"Get the list of users in the current domain"
1. Get-DomainUser
2. Get-DomainUser -Identity username
```

###### Finding properties for users in the current domain

```powershell title:"Get the user and its properties"
Get-DomainUser -Identity username -Properties *
Get-DomainUser -Properties samaccountname, logonCount
```

###### Finding computers in the current domain

```powershell title:"Get a list of computers in the current domain"
Get-DomainComputer | select Name
Get-DomainComputer -OperatingSystem "*Server 2022*"
Get-DomainComputer -Ping
```

```powershell title:"Computers"
Get-NetComputer
```

###### Finding groups in the current domain

```powershell title:"Get all the groups in the current domain"
Get-DomainGroup | select Name
Get-DomainGroup -Domain <domain.name>
```

```powershell title:"Get all groups containing the word "admin" in the group name"
Get-DomainGroup *admin*
```

```powershell title:"Get the SID of the Enterprise Admins group"
Get-DomainGroup -Identity *Enterprise Admins* -Domain domain.com -Properties objectsid
```

###### Finding members of a group

```powershell title:"Get all the members of the Domain Admins group"
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

```powershell title:"Look for machines where the current user is a local admin or a Domain Admin"
Find-DomainUserLocation
```

```powershell title:"Get the group membership for a user"
Get-DomainGroup -UserName "whatever"
```

###### Finding local groups on a machine

```powershell title:"Get all the local groups on a machine (needs administrator privileges on non-dc machines)"
Get-NetLocalGroup -Computername fqdn\computer-name
```

```powershell title:"Get members of the local group "Administrators" on a machine (needs administrator privileges on non-dc machines)
Get-NetLocalGroupMember -ComputerName fqdn\computer-name -GroupName Administrators
```

###### Finding actively logged users on a computer for OPSEC

```powershell title:"Get actively logged users on a computer (needs local administrator rights on the target)"
Get-NetLoggedon -ComputerName fqdn\computer-name
```

```powershell title:"Get locally logged users on a computer (needs remote registry enabled on the target - started by-default on server OS)"
Get-LoggedonLocal -ComputerName fqdn\computer-name
```

```powershell title:"Get the last logged user on a computer (needs adminstrative rights and remote registry enabled on the target)"
Get-LastLoggedOn -ComputerName computer-name
```

###### Finding shares on hosts in the current domain

```powershell title:"Find shares on hosts in the current domain"
Find-DomainShare
```

```powershell title:"Find shares on hosts in the current domain (faster)"
Invoke-ShareFinder -Verbose
```

```powershell title:"Find sensitive files on computers in the domain"
Invoke-FileFinder -Verbose
```

```powershell title:"Get all fileservers of the domain"
Get-NetFileServer
```
###### Finding GPOs and related information in the current domain

```powershell title:"Get list of GPOs in the current domain"
1. Get-DomainGPO
2. Get-DomainGPO -ComputerIdentity machine-name
```

```powershell title:"Get GPO(s) which use Restricted Groups or groups.xml for interesting users"
Get-DomainGPOLocalGroup
```

```powershell title:"Get users which are in a local group of a machine using GPO"
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity machine-name
```

```powershell title:"Get machines where the given user is a member of a specific group"
Get-DomainGPOUserLocalGroupMapping -Identity username -Verbose
```

###### Finding OUs in a domain

```powershell title:"Get OUs in a domain"
Get-DomainOU
```

```powershell title:"Get GPO applied on an OU. Read GPOname from gplink from Get-NetOU"
Get-DomainGPO -Identity "{GPO here}"
```

###### Finding Access Control Lists of objects

```powershell title:"Get the ACLs associated with the specified object"
Get-DomainObjectAcl -SamAccountName username -ResolveGUIDs
```

```powershell title:"Get the ACLs associated with the specified prefix to be used for search"
Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=namebeforefirstdot,DC=namebeforeseconddot,DC=namebeforeanymoredots" -ResolveGUIDs -Verbose
```

```powershell title:"Search for interesting ACEs"
Find-InterestingDomainAcl -ResolveGUIDs
```

```powershell title:"Get the ACLs associated with the specified path"
Get-PathAcl -Path "\\computer-name\sysvol"
```


