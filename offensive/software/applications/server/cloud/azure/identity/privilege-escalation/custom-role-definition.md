---
title: "Custom role definition: a wildcard role you assign yourself"
description: "Crafting a custom role carrying Microsoft.Authorization/*/write or wildcard actions through roleDefinitions/write, then self-assigning it."
keywords:
  - custom role
  - roleDefinitions write
  - wildcard actions
  - Azure RBAC
  - privilege escalation
---

# Custom role definition

`Microsoft.Authorization/roleDefinitions/write` lets you create or edit a custom role. If you hold it without full Owner, you craft a role whose `actions` include `Microsoft.Authorization/*/write` (the right to assign roles) or a bare `*`, then assign that role to yourself and become able to grant anything.

## Defining and assigning the role

```bash
cat > role.json <<'EOF'
{ "Name": "support-helper",
  "IsCustom": true,
  "Actions": ["*"],
  "AssignableScopes": ["/subscriptions/<sub-id>"] }
EOF
az role definition create --role-definition role.json
az role assignment create --assignee <your-object-id> \
  --role "support-helper" --scope /subscriptions/<sub-id>
```

An innocuous name (`support-helper`, `backup-reader`) and a narrow `AssignableScopes` draw less attention than a role literally named admin.

## Exploitation notes

- `Microsoft.Authorization/*/write` alone is enough: it grants `roleAssignments/write`, so you do not need a bare `*` to then assign yourself Owner.
- Editing an existing custom role you are already assigned (add actions to it) is quieter than creating a new one and a new assignment.
- The role is only usable within its `AssignableScopes`; set it to the highest scope you can write.

## Tools

- **az cli** (`az role definition create` / `update`).
- **BARK**: enumerate custom-role edit rights and abuse them from a token.

## References

- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Azure custom roles](https://learn.microsoft.com/azure/role-based-access-control/custom-roles)
- [HackTricks Cloud: Azure RBAC privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
