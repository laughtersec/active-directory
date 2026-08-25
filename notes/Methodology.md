---
tags:
  - general
---
![[methodology.svg ]]


- It is supposed to start off with reconnaissance, but we are assuming a position where we already have a foothold.
- Not much can be done without enumeration, which will mostly rely on the usage of BloodHound
- Most of the persistence techniques and some enumeration can't be done without higher privileges, so we need to find ways to escalate privileges locally. There might be admin sessions open on other machines which is worth checking out. Again, all thanks to LPE.
- As an administrator on a machine, we can look for many other interesting things which will allow us to move laterally, get domain admin or move to other forests that trust the forest or the machine in the forest that we have assumed to be compromised.
- While this is only required in a real red-team assessment and not in the exam, **persistence** allows attackers to stay in the network as long as they wish. Opsec must be taken into account and decoys must be avoided.