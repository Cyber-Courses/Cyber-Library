---
title: "Reconnaissance: mapping the Active Directory domain"
description: "Reading Active Directory from any foothold to map the domain: hosts and domain layout, LDAP objects and attributes, users and groups, logon sessions, password policy, and attack paths with BloodHound."
keywords:
  - active directory enumeration
  - LDAP
  - bloodhound
  - domain recon
  - session enumeration
---

# Reconnaissance

Almost everything in Active Directory is readable by any authenticated account, and much of it from an unauthenticated network position. Before attacking authentication, you read the domain to find what to attack: which accounts have weak or roastable credentials, which permissions are delegable, where privileged users are logged on, and how domains trust each other. This reconnaissance feeds every other technique, so it comes first.

## What you are mapping

- **The domain and its hosts**: domain controllers, naming contexts, functional levels, and reachable machines.
- **Objects and attributes**: users, computers, groups, and the LDAP attributes that reveal SPNs, delegation flags, and secrets left in fields.
- **Sessions**: where users, especially admins, are currently logged on, for targeting.
- **Policy**: the password and lockout policy that bounds spraying.
- **Attack paths**: the graph of permissions and relationships that BloodHound turns into a route to Domain Admin.

## Pages

- **[Host and domain discovery](host-and-domain-discovery.md)**: finding domain controllers, naming contexts, and the lay of the domain.
- **[LDAP enumeration](ldap-enumeration.md)**: querying the directory for objects and attributes.
- **[BloodHound](bloodhound.md)**: collecting and graphing attack paths.
- **[User and group enumeration](user-and-group-enumeration.md)**: enumerating principals and memberships.
- **[Session enumeration](session-enumeration.md)**: locating logged-on users for targeting.
- **[Password policy](password-policy.md)**: reading the policy that bounds safe spraying.

## References

- The Hacker Recipes: Active Directory recon
- Microsoft: Active Directory Domain Services documentation
