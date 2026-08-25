---
tags:
  - detection-and-defense
title:
  - Detection
---
## By Event ID

#### [[golden_ticket]]
- 4624: Account Logon
- 4672: Admin Logon

```powershell title:"Look for 4672s"
Get-WinEvent -Filterhashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List -Property *
```

#### [[Silver Ticket]]
- 4624: Account Logon
- 4634: Account Logoff
- 4672: Admin Logon

```powershell title:"Look for 4672s"
Get-WinEvent -Filterhashtable @{Logname='Security';ID=4672} -MaxEvents 1 | Format-List -Property *
```

#### [[Skeleton Key]]
##### Events
- System Event ID 7045 - A service was installed on the system. (Type Kernel Mode driver)
##### Events ("Audit privilege use" must be enabled)
- Security Event ID 4673 - Sensitive Privilege Use
- Event ID 4611 - A trusted logon process has been registered with the Local Security Authority (LSA)

```powershell title:"Look for 7045s"
Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*"}
```

```powershell title:"Look for 7045s (not recommended method)"
Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*" -and $_.message -like "*mimidrv*"}
```
##### Mitigation
- Running lsass.exe as a protected process because it would force an attacker to load a kernel mode driver.
- Should be tested thoroughly as many drivers and plugins might not load with the protection.

```powershell title:"Make registry changes to run lsass as a protected process"
New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name RunAsPPL -Value 1 -Verbose
```

```powershell title:"Verify lsass.exe is running as a protected process after rebooting"
Get-WinEvent -FilterHashTable @{Logname='System';ID=12} | ?{$_.message -like "*protected process*"}
```

#### [[dsrm]]
- 4657 - Audit creation/change of `HKLM:\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior`

#### [[custom_ssp]]
- 4657 - Audit creation/change of `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages`

#### [[Kerberoasting]]
##### Events
- 4769 - A Kerberos ticket was requested

Since 4769 is logged very frequently on a DC, we may like to filter results based on the following information from logs:
- Service name should not be krbtgt
- Service name does not end with `$` (to filter out machine accounts used for services)
- Account name should not be machine@domain (to filter out requests from machines)
- Failure code is '0x0' (to filter out failures, 0x0 is success)
- Ticket encryption type is 0x17
##### Mitigation
- Service Account Passwords should be hard to guess (greater than 35 characters)
- Use Group Managed Service Accounts (Automatic change of password periodically and delegated SPN Management)

#### ACL Attacks
##### Events
- 4662 (Audit Policy object must be enabled) - An operation was performed on an object
- 5136 (Audit Policy object must be enabled) - A directory service object was modified
- 4670 (Audit Policy for object must be enabled) - Permissions on an object were changed

A useful tool - [AD ACL Scanner](https://github.com/canix1/ADACLScaner) - Create and compare create reports of ACLs





