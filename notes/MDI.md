---
tags:
  - detection-and-defense
title:
  - MDI
---
- It identifies, detects and investigates advanced threats, compromised identities, and malicious insider actions directed at your organization
- MDI sensors are installed on DCs and Federation servers. Analysis and alerting is done in the Azure cloud.
- MDI can be used for detecting
	- Recon
	- Compromised credentials (Brute-Force, [[kerberoasting]] etc.)
	- Lateral Movement (PTH, OPTH etc.)
	- Domain Dominance (DCSync, [[golden_ticket]], [[skeleton_key]] etc.)
	- Exfiltration
## Bypass
- The key is to avoid talking to the DC as long as possible and make the traffic that we generate appear benign.
- To bypass DCSync detection, target users which are whitelisted.
- If we have NTLM hash of a DC, we can extract NTLM hashes of any machine account using netsync.

#### [[golden_ticket]]
- If we forge a Golden ticket with SID History of the Domain Controllers group and Enterprise Domain Controllers Group, there are less chances of detection by MDI

```batch
BetterSafetyKatz.exe "kerberos::golden /user:domain-controller-machine-account /id:1000 /sid:S-1-XXXXX /sids:S-1-XXXXX,S-1-XXX /krbtgt:<ntlm_hash> /ptt" "exit"
```

