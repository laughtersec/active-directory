---
tags:
  - defense
cssclasses:
---
- System-wide transcription
- Script-Block logging
- AntiMalware Scan Interface (AMSI)
- Constrained Language Mode (CLM) integrated with AppLocker WDAC ([[Device Guard]])
### Trust Tickets
##### SID Filtering
- Avoid attacks which abuse SID history
- Enabled by default on all inter-forest trusts. Intra-forest trusts are assumed secured by default (MS considers forest and not the domain to be a security boundary).
- But, since SID filtering has potential to break applications and user access, it is often disabled.
##### Selective Authentication
- In an inter-forest trust, if Selective Authentication is configured, users between the trusts will not be automatically authenticated. Individual access to domains and servers in the trusting domain/forest should be given.

### Deception
- Adversary mindset of going for the "lowest hanging fruit" and illusive superiority over defenders.
- We must provide the adversaries what they are looking for, for instance:
	- A user with high privileges
	- Permissions over other objects
	- Poorly configured ACLs
	- Misconfigured/dangerous user attributes etc.
- We can use [Deploy-Deception](https://github.com/samratashok/Deploy-Deception)
- `Windows Settings > Security Settings > Advanced Audit Policy Configuration > DS Access > Audit Directory Service Access Group Policy` needs to be configured to enable 4662 logging.

```powershell title:"Create a decoy user whose password never expires and a 4662 is logged whenever x500uniqueIdentifier xxxx property of the user is read"
Create-DecoyUser -UserFirstName user -UserLastName manager -Password Pass@123 | Deploy-UserDeception -UserFlag PasswordNeverExpires -GUID d07da11f-8a3d-42b6b0aa-76c962be719a -Verbose
```

This property is not read by net.exe, WMI classes (like Win32_UserAccount) and [[ActiveDirectory Module]]. But LDAP based tools like PowerView and ADExplorer trigger the logging.

```powershell title:"Create a decoy user and make it a mamber of the Domain Admins group. As a protection against potential abuse, deny logon to the user on any machine"
Create-DecoyUser -UserFirstName dec -UserLastName da -Password Pass@123 | Deploy-PrivilegedUserDeception -Technique DomainAdminsMembership -Protection DenyLogon -Verbose
```

- If there is any attempt to use the user credentials (password or hashes), a 4768 is logged.
- Any enumeration which reads DACL or all properties for the user will result in 4662 logging.