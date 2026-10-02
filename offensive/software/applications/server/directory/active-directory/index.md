---
title: "Active Directory: attacking the Windows domain and its authentication ecosystem"
description: "The Active Directory attack surface organized by mechanism: enumeration, authentication and credential abuse (NTLM, Kerberos, certificates), DACL abuse, Group Policy, trusts, and persistence."
keywords:
  - active directory
  - kerberos
  - ntlm
  - DACL
  - domain compromise
  - AD CS
---

# Active Directory

Active Directory (AD) is the identity and authorization backbone of most Windows enterprises: a replicated, LDAP-accessible database of users, computers, groups, and policies, authenticated with Kerberos and NTLM and increasingly with certificates. An attacker almost never needs a software vulnerability to take it over. The domain is compromised by **abusing its intended features**: readable object attributes, delegable permissions, crackable service tickets, relayable authentication, and over-permissive access-control entries.

## How this area is organized

After **enumeration**, AD is organized by the **mechanism being abused**, because that is the durable primitive and how a technique is looked up (Kerberoasting is a Kerberos technique regardless of which engagement phase you use it in). The heart of the tree is a single **authentication and credentials** section, since passwords, NTLM hashes, Kerberos tickets, and certificates are all just different forms of the same thing, authentication material you obtain and reuse. Permission-based privilege escalation (DACLs, Group Policy), lateral trust abuse, and domain persistence follow.

## The attack arc

A typical path, which these sections support in order:

1. **Enumerate** the domain from any authenticated (often any network) position to map users, groups, computers, delegations, ACLs, and trusts.
2. **Obtain authentication material**: spray or roast for crackable secrets, dump credentials from a foothold, relay coerced authentication, or abuse certificate templates.
3. **Escalate** by abusing object permissions (DACLs) and Group Policy that let a controlled principal rewrite privileged objects.
4. **Cross boundaries** by abusing domain and forest trusts.
5. **Persist** with forged tickets, replication abuse, and privileged-object backdoors.

## Sections

- **[Enumeration](enumeration/index.md)**: mapping the domain, its objects, permissions, and trusts.
- **Authentication and credentials**: NTLM, Kerberos, certificates (AD CS), and credential dumping and cracking.
- **DACL abuse**: abusing object permissions to control privileged principals.
- **Group Policy**: abusing GPOs to run code and change configuration.
- **Trusts**: intra-forest and cross-forest trust abuse.
- **Built-ins and settings**: default quotas, legacy settings, and privileged groups.
- **Persistence**: maintaining domain control after compromise.

## References

- The Hacker Recipes: Active Directory
- Microsoft: Active Directory Domain Services documentation
