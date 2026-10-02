---
title: "Reconnaissance: mapping the Active Directory domain"
description: "Reading Active Directory from any foothold to map the domain before attacking it: hosts and domain layout, LDAP objects and attributes, logged-on sessions, and the attack-path graph with BloodHound."
keywords:
  - active directory enumeration
  - LDAP
  - bloodhound
  - domain recon
  - session enumeration
---

# Reconnaissance

Almost everything in Active Directory is readable by any authenticated account, and much of it from an unauthenticated network position. Before attacking, you read the domain to decide what to attack: which hosts and domains exist, what the directory exposes about users, computers, and delegation, where privileged users are logged on, and how permissions chain into a path to Domain Admin. This general reconnaissance feeds every other technique, which is why it stands on its own; the enumeration specific to one surface (service accounts, ACLs, GPOs, trusts) lives with that surface.

## What you are mapping

- **The domain and its hosts**: domain controllers, naming contexts, functional levels, and reachable machines.
- **Objects and attributes**: users, computers, and groups, and the LDAP attributes that leak SPNs, delegation flags, and secrets left in fields.
- **Sessions**: where users, especially admins, are currently logged on, for targeting.
- **Attack paths**: the graph of permissions and relationships that BloodHound turns into a route to Domain Admin.

## Pages

- **[Host and domain discovery](host-and-domain-discovery.md)**: finding domain controllers, naming contexts, and the lay of the domain.
- **[LDAP enumeration](ldap-enumeration.md)**: querying the directory for objects and attributes.
- **[BloodHound](bloodhound.md)**: collecting and graphing attack paths.
- **[Session enumeration](session-enumeration.md)**: locating logged-on users for targeting.

## References

- [Microsoft: Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services)
- [SpecterOps: BloodHound documentation](https://bloodhound.specterops.io/)
- [NetExec: LDAP and SMB enumeration](https://github.com/Pennyw0rth/NetExec)
