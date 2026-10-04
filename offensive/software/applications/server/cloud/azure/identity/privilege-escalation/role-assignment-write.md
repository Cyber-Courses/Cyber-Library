---
title: "Role assignment write: granting yourself Owner"
description: "Assigning yourself or a controlled principal Owner or Contributor with Microsoft.Authorization/roleAssignments/write at resource, group, subscription, or management-group scope."
keywords:
  - roleAssignments write
  - Owner
  - Contributor
  - Azure RBAC
  - scope
  - Microsoft.Authorization
---

# Role assignment write

`Microsoft.Authorization/roleAssignments/write` is the permission to grant RBAC roles. A principal that holds it (through Owner, User Access Administrator, or a custom role) can assign itself, or a principal it controls, a higher role at any scope it reaches: a single resource, a resource group, a subscription, or a whole management group.

## Granting Owner

```bash
# subscription-wide Owner to yourself
az role assignment create --assignee <your-object-id> \
  --role Owner --scope /subscriptions/<sub-id>

# or scope it to a resource group / single resource
az role assignment create --assignee <object-id> --role Contributor \
  --scope /subscriptions/<sub>/resourceGroups/<rg>
```

A service principal you control is a quieter grantee than your own user, and survives the user being disabled.

## Exploitation notes

- The highest scope you can write at is what matters: `roleAssignments/write` at a management group grants down to every subscription beneath it.
- Owner includes `roleAssignments/write`; Contributor does not, so Contributor cannot grant itself Owner. That boundary is why [elevate-access](elevate-access.md) and [custom roles](custom-role-definition.md) matter when you only hold Contributor-like rights.
- Assignments are visible in the activity log and the portal; a scoped assignment on an obscure resource group draws less attention than a subscription Owner grant.

## Tools

- **az cli** (`az role assignment create`).
- **MicroBurst** (`Invoke-AzureRmWebAppsExfil`-era modules enumerate and abuse RBAC).
- **BARK** (`New-AzureRMRoleAssignment`): scripted assignment from a token.

## References

- [SpecterOps: abusing Azure RBAC with BARK](https://github.com/BloodHoundAD/BARK)
- [Microsoft: Azure built-in roles](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles)
- [HackTricks Cloud: Azure RBAC privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
