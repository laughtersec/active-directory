---
tags:
  - detection-and-defense
cssclasses:
---
- [Windows Defender Application Control](https://learn.microsoft.com/en-us/windows/security/identity-protection/credential-guard/) (WDAC) is a group of features designed to harden a system against malware attacks. Its focus is preventing malicious code from running by ensuring only known good code can run.
- Primary components -
	- Configurable Code Integrity (CCI) - Configure only trusted code to run.
	- Virtual Secure Mode Protected Code Integrity - Enforces CCI with Kernel mode (KMCI) and User mode (UMCI)
	- Platform and UEFI Secure Boot - Ensures boot binaries and firmware integrity.
- UMCI is something which interferes with most of the lateral movement attacks we have seen.
- While it depends on the deployment (discussing which will be too lengthy), many well known application whitelisting bypasses - signed binaries like csc.exe, MSBuild.exe etc. - are useful for bypassing UMCI as well.
- See [LOLBAS](https://lolbas-project.github.io) project
