---
tags:
  - local-privesc
cssclasses:
---
 [[PowerShell]] LPE Module
#### Services issues using PowerUp

```powershell title:"Get services with unquoted paths and a space in their name"
Get-ServiceUnquoted -Verbose
```

```powershell title:"Get services where the current user can write to its binary path or change arguments to the binary"
Get-ModifiableServiceFile -Verbose
```

```powershell title:"Get the services whose configuration current user can modify"
Get-ModifiableService -Verbose
```

