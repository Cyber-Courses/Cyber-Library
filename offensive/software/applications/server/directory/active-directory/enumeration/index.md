---
title: "Active Directory enumeration: mapping the domain, objects, permissions, and trusts"
description: "Reconnaissance of an Active Directory domain: discovering domain controllers and naming contexts, querying objects over LDAP, graphing attack paths with BloodHound, and enumerating users, SPNs, ACLs, GPOs, sessions, trusts, and password policy."
keywords:
  - active directory enumeration
  - LDAP
  - BloodHound
  - SPN
  - ACL enumeration
  - domain recon
---

# Enumeration

Enumeration is the foundation of every Active Directory attack. AD is a read-heavy service: by design, almost any authenticated principal (and often an unauthenticated one) can read the bulk of the directory over LDAP, which hands an attacker a near-complete map of users, computers, groups, delegations, permissions, and trust relationships. The quality of that map decides which of the later attacks are even reachable, so thorough enumeration comes first.

## What you are building

A picture of the domain detailed enough to pick attacks:

- **The terrain**: domain and forest names, domain controllers, sites, and the LDAP naming contexts that anchor every query.
- **The principals**: users, computers, and groups, who is privileged, which accounts are service accounts, and which are stale or mis-set.
- **The permissions**: the access-control entries on objects that let one principal rewrite another, which are the raw material for privilege escalation.
- **The reach**: group memberships, Group Policy scope, logged-on sessions, and domain and forest trusts that define lateral and cross-boundary movement.

## Collect once, analyze repeatedly

Modern practice is to collect the directory once and analyze it offline as a graph rather than running ad-hoc queries. BloodHound is the standard for this: it ingests users, groups, ACLs, sessions, and trusts and computes attack paths to a target such as Domain Admins. The pages below cover both the targeted queries and the bulk collection that feeds that graph.

## Pages

- **[Host and domain discovery](host-and-domain-discovery.md)**: finding domain controllers, the domain and forest, and LDAP naming contexts.
- **[LDAP enumeration](ldap-enumeration.md)**: querying AD objects directly over LDAP with filters.
- **[BloodHound](bloodhound.md)**: bulk collection and attack-path graphing.
- **[User and group enumeration](user-and-group-enumeration.md)**: users, groups, RID cycling, and valid-user discovery.
- **[SPN discovery](spn-discovery.md)**: locating service accounts for roasting.
- **[ACL enumeration](acl-enumeration.md)**: finding abusable access-control entries.
- **[GPO and OU enumeration](gpo-and-ou-enumeration.md)**: Group Policy objects, their links, and scope.
- **[Session enumeration](session-enumeration.md)**: who is logged on where, for targeting.
- **[Trust enumeration](trust-enumeration.md)**: mapping domain and forest trusts.
- **[Password policy](password-policy.md)**: thresholds for safe spraying.

## References

- The Hacker Recipes: Active Directory reconnaissance
- SpecterOps: BloodHound documentation
