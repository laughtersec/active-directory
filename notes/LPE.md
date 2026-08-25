---
tags:
  - local-privesc
---
There are various ways of locally escalating privileges on Windows

- Missing patches
- Automated deployment and AutoLogon passwords in clear text
- AlwaysInstallElevated (Any user can run MSI as SYSTEM)
- Misconfigured Services
- DLL Hijacking
- NTLM Relaying a.k.a Won't Fix

#### Tools for complete coverage:

- [[PowerUp]] - https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc

```powershell title:"Usage"
Invoke-AllChecks
```

- Privesc - https://github.com/enjoiz/Privesc

```powershell title:"Usage"
Invoke-PrivEsc
```

- winPEAS - https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS

```powershell title:"Usage"
winPEASx64.exe
```

- NTLM Relaying example - https://github.com/antonioCoco/RemotePotato0

