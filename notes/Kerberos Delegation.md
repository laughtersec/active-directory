---
tags:
  - privesc
cssclasses:
---
- Kerberos Delegation allows to "reuse the end-user credentials to access resources hosted on a different server".
- This is typically useful in multi-tier service or applications where Kerberos Double Hop is required.
- For example, users authenticates to a web server and a web server makes requests to a database server. The web server can request access to resources (all or some resources depending on the type of delegation) on the database server as the user and not as the web server's service account.
- The service account for the web service must be trusted for delegation to be able to make requests as a user.
![[Kerberos_Delegation.svg]]
1. A user provides credentials to the Domain Controller
2. The DC returns a TGT
3. The user requests a TGS for the web service on Web Server
4. The DC provides a TGS
5. The user sends the TGT and TGS to the web server
6. The Web server service account uses the user's TGT to request a TGS for the database server from the DC
7. The Web server service account connects to the database server as the user

There are two types of Kerberos Delegation:

- General/Basic or [[Unconstrained Delegation]] - allows the first hop server (web server in our example) to request access to any service on any computer in the domain.
- [[Constrained Delegation]] - allows the first hop server (web server in our example) to request access only to specified services on specified computers. If the user is not using Kerberos authenticate to the first hop server, Windows offers Protocol Transition to transition the request to Kerberos.

*In both types of delegations, a mechanism is required to impersonate the incoming user and authenticate to the second hop server (database server in our example) as the user.*

