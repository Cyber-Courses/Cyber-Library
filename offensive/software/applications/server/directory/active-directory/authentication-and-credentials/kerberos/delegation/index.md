---
title: "Delegation: abusing Kerberos impersonation"
description: "Abusing Kerberos delegation in Active Directory: unconstrained delegation TGT capture, constrained delegation with protocol transition (S4U2self/S4U2proxy), and resource-based constrained delegation configured from a writable computer object."
keywords:
  - kerberos delegation
  - unconstrained delegation
  - constrained delegation
  - RBCD
  - S4U
---

# Delegation

Kerberos delegation lets a service act **on behalf of** a user toward another service, so a front-end (a web app, say) can reach a back-end (a database) as the user. The feature necessarily lets one account obtain tickets for another identity, and every delegation type is therefore an impersonation primitive when the delegating account is compromised or its configuration is writable.

## The three types

- **Unconstrained**: the delegating host receives and stores the user's **TGT**, so it can act as that user *anywhere*. Compromise such a host (or coerce a DC to authenticate to it) and you capture TGTs, including a DC's.
- **Constrained**: the account may request tickets for a user only to a **listed set of SPNs**, using the S4U extensions, often with **protocol transition** that lets it do so without the user ever authenticating.
- **Resource-based constrained (RBCD)**: the permission lives on the **target** object (`msDS-AllowedToActOnBehalfOfOtherIdentity`), so write access to a computer object lets you nominate an account that may impersonate anyone to it.

## The S4U mechanism

Constrained and resource-based delegation both run on the **Service-for-User** extensions:

- **S4U2self**: the service asks the KDC for a ticket to *itself* as an arbitrary user (no user involvement). This yields an impersonation ticket.
- **S4U2proxy**: the service uses that ticket to request a ticket to the *back-end* service as the user.

Whether the resulting ticket is **forwardable** (and so usable onward) depends on protocol transition and account flags, which is the crux of what each abuse can reach.

## Pages

- **[Unconstrained delegation](unconstrained.md)**: capturing TGTs, including via coercion.
- **[Constrained delegation](constrained.md)**: S4U2self/S4U2proxy and protocol transition abuse.
- **[Resource-based constrained delegation](resource-based-constrained.md)**: writing the delegation attribute to impersonate.

## References

- The Hacker Recipes: Kerberos delegations
- Microsoft: S4U2self, S4U2proxy, and delegation
