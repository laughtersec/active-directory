---
tags:
  - domain-enumeration
---
The default [[PowerShell]] module for enumerating objects in [[Active Directory]]

```powershell title:"Importing"
. .\Microsoft.ActiveDirectory.Management.dll
Import-Module Microsoft.ActiveDirectory.Management.dll
```

```powershell title:"Get the current domain"
Get-ADDomain
```

```powershell title:"Get object of another domain"
Get-ADDomain -Identity domain.com
```

```powershell title:"Get domain SID for the current domain"
(Get-ADDomain).DomainSID
```

```powershell title:"Get domain controllers for the current domain"
Get-ADDomainController
```

```powershell title:"Get domain controllers for another domain"
Get-ADDomainController -DomainName domain.com -Discover
```

```powershell title:"Get a list of users in the current domain"
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity user1 -Properties *
```

```powershell title:"Get list of all properties for users in the current domain"
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```

```powershell title:"Search for a particular string in a user's attributes"
Get-ADUser -Filter 'Description -like "*example*"' -Properties Description | select name,Description
```

```powershell title:"Get a list of computers in the current domain"
Get-ADComputer -Filter * | select Name
Get-ADComputer -Filter * -Properties *
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```

```powershell title:"List all the groups in the current domain"
Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties *
```

```powershell title:"List all the groups containing the word "admin" in group name"
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```

```powershell title:"Get all members of Domain Admins group"
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

```powershell title:"Get group membership for a user"
Get-ADPrincipalGroupMembership -Identity user1
```

```powershell title:"Get OUs in a domain"
Get-ADOrganizationalUnit -Filter * -Properties *
```

```powershell title:"Get a list of all domain trusts for the current domain"
Get-ADTrust
Get-ADTrust -Identity domain.com
```

```powershell title:"Get details about the current forest"
Get-ADForest
Get-ADForest -Identity forest.com
```

```powershell title:"Get all domains in the current forest"
(Get-ADForest).Domains
```

```powershell title:"Get all global catalogs for the current forest"
Get-ADForest | select -ExpandProperty GlobalCatalogs
```

```powershell title:"Map trusts of a forest"
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
```




