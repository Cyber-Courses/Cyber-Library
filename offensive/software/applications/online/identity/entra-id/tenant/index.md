---
title: "Entra tenant"
description: "Crossing Entra tenant boundaries: B2B guest access and cross-tenant access policies that expose resources to external identities."
keywords:
  - tenant
  - B2B
  - guest access
  - cross-tenant access
  - external identity
---

# Tenant

The tenant boundary is meant to separate organizations, but B2B collaboration and cross-tenant access settings punch holes in it. A guest account is a foothold inside a tenant you are external to, and loose cross-tenant trust can let claims (MFA, device compliance) or synchronization from one tenant carry into another.

## What folds in here

- **[Guest access](guest-access.md)**: B2B invitation, guest enumeration, and guest-to-member escalation.
- **[Cross-tenant access](cross-tenant-access.md)**: inbound trust and cross-tenant synchronization abuse.

## References

- [dirkjanm.io: cross-tenant and B2B research](https://dirkjanm.io/)
- [HackTricks Cloud: Entra tenant](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: cross-tenant access settings](https://learn.microsoft.com/entra/external-id/cross-tenant-access-overview)
