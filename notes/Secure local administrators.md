---
tags:
  - defense
---
#### LAPS (Local Administrator Password Solution)
- Centralized storage of passwords in AD with periodic randomizing where read permissions are access controlled.
- Computer objects have two new attributes: `ms-mcs-AdmPwd `attribute stores the clear text password and `ms-mcs-AdmPwdExpirationTime` controls the password change.
- Storage in clear text, transmission is encrypted.
- Note - With careful enumeration, it is possible to retrieve which users can access the clear text password providing a list of attractive targets.
