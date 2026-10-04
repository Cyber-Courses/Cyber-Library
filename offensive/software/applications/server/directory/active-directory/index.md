---
title: "Active Directory: attacking the Windows domain and its authentication ecosystem"
description: "The Active Directory attack surface organized by what is attacked: domain reconnaissance, authentication and credentials (NTLM, Kerberos, certificates), DACL, Group Policy, trusts, and managed accounts."
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

This area is organized by the **attack surface**, the thing being abused, because that is the durable primitive and how a technique is looked up: Kerberoasting is a Kerberos technique regardless of which engagement phase you use it in. Each topic covers its own lifecycle, from finding it to abusing it to persisting through it, rather than splitting those into separate kill-chain sections. General **reconnaissance**, reading the directory to find what to attack, comes first as its own surface. The largest surface by far is then **authentication**, since passwords, NTLM hashes, Kerberos tickets, and certificates are all forms of the same thing, authentication material you obtain and reuse. Permission-based escalation (DACLs, Group Policy), cross-boundary trust abuse, and the accounts AD manages for you are their own surfaces.

## The attack arc

A typical path cuts across these surfaces in order:

1. **Reconnaissance**: read the domain from any authenticated (often any network) position to map users, groups, computers, delegations, ACLs, and trusts.
2. **Obtain authentication material**: spray or roast for crackable secrets, dump credentials from a foothold, relay coerced authentication, or abuse certificate templates.
3. **Escalate** by abusing object permissions (DACLs) and Group Policy that let a controlled principal rewrite privileged objects.
4. **Cross boundaries** by abusing domain and forest trusts.
5. **Persist** with forged tickets, replication abuse, and privileged-object backdoors, covered within each surface it belongs to.

## Topics

- **[Authentication](authentication/index.md)**: credential dumping and cracking, NTLM, Kerberos, and certificates (AD CS).
- **[DACL](dacl/index.md)**: abusing object permissions to control privileged principals.
- **[Group Policy](group-policy/index.md)**: abusing GPOs to run code and change configuration.
- **[Trusts](trusts/index.md)**: intra-forest and cross-forest trust abuse.
- **[Managed accounts](managed-accounts/index.md)**: gMSA and LAPS secrets in the directory, and dMSA (BadSuccessor) escalation.

## References

- The Hacker Recipes: Active Directory
- Microsoft: Active Directory Domain Services documentation
