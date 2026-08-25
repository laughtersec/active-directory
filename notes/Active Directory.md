---
tags:
  - general
---
##### Introduction
- Active Directory is a directory service developed by Microsoft for Windows domain networks.
- Windows Server operating systems include it as a set of processes and services.
- Originally, only centralized domain management used Active Directory. However, it ultimately became an umbrella title for various directory-based identity-related services.
- Now there is complete control of not just the domains but also the users, groups and machines inside an active directory network. Every single thing that is part of an organization's IT infrastructure can be managed, and controlled.
##### Organizational Unit (OU)
- Building blocks of a domain including computers, users, groups etc.
- The AD catalog consists of several objects with different attributes which exists for different purposes. You would categorize them using OUs.
##### Forest
- Security boundary
- May contain multiple domains and those domains may contain multiple OUs
##### Group Policy Origin (GPO)
- Provides the ability to manage configuration and changes easily and centrally in AD.
- Allows configuration of:
	- Security Settings
	- Registry-based policy settings
	- Group policy preferences like startup/shutdown/log-on/logoff scripts settings
	- Software installation
- GPO can be abused for various attacks like privesc, backdoors, persistence etc.

