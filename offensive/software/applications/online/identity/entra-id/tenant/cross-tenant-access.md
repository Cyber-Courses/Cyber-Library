---
title: "Cross-tenant access: trusting another tenant's claims"
order: 2
description: "Abusing cross-tenant access settings and B2B trust: inbound trust of MFA and compliant-device claims and cross-tenant synchronization."
keywords:
  - cross-tenant access
  - B2B trust
  - inbound trust
  - cross-tenant sync
  - multi-tenant
---

# Cross-tenant access

Cross-tenant access settings decide how one tenant trusts identities and claims from another. When a tenant **inbound-trusts** another tenant's MFA or compliant-device claims, compromising the trusted tenant lets you satisfy the target's conditional access with claims minted in the tenant you control. Cross-tenant synchronization can even provision accounts from one tenant into another.

## Abuse the trust

```bash
# read the target's cross-tenant access policy (as an authenticated principal)
az rest --method GET --url "https://graph.microsoft.com/v1.0/policies/crossTenantAccessPolicy/partners"
# if inbound trust accepts the home tenant's MFA/compliant-device claims, a token from the
# controlled tenant satisfies the target's device/MFA conditional access
```

## Exploitation notes

- Inbound trust of MFA means you do not re-MFA from the trusted tenant: own that tenant and its claims pass into the target.
- Cross-tenant synchronization provisions users from a source tenant into the target, a path to a standing account if you control the source.
- B2B direct connect shares resources (for example Teams shared channels) across the trust, widening reachable data.

## Tools

- **az cli** / **Graph** (`crossTenantAccessPolicy`).
- **ROADtools**: cross-tenant policy enumeration.

## References

- [dirkjanm.io: cross-tenant trust research](https://dirkjanm.io/)
- [HackTricks Cloud: cross-tenant access](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: cross-tenant access settings](https://learn.microsoft.com/entra/external-id/cross-tenant-access-overview)
