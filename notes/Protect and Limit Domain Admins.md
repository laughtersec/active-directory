---
tags:
  - defense
---
- Reduce the number of Domain Admins in your environment
- Do not allow or limit login of DAs to any other machine other than the Domain Controllers. If logins to some servers is necessary, do not allow other administrators to login to that machine.
- (Try to) Never run a service with a Domain Admin.
- Set "Account is sensitive and cannot be delegated" for Domain Admins.

#### Protected Users Group
- Protected Users is a group introduced in Server 2012 R2 for "better protection against credential theft" by not caching credentials in insecure ways. A user added to this group has the following major device protections:
	- Cannot use CredSSP and WDigest - No more cleartext credentials caching.
	- NTLM hash is not cached.
	- Kerberos does not use DES or RC4 keys. No caching of clear text credentials or long term keys.
- If the domain functional level is Server 2012 R2, following DC protections are available:
	- No NTLM authentication
	- No DES or RC4 keys in Kerberos pre-auth
	- No delegation (constrained or unconstrained)
	- No renewal of TGT beyond initial four hour lifetime - Hardcoded, unconfigurable "Maximum lifetime for user ticket" and "Maximum lifetime for user ticket renewal"
- Needs all domain control to be at least Server 2008 or later (because AES keys)
- Not recommended by MS to add Domain Admins and Enterprise Admins to this group without testing "the potential impact" of lock out.
- No cached logon i.e. no offline sign-on.
- Having computer and service accounts in this group is useless as their credentials will always be present on the host machine.