---
title: "Élévation de privilèges : des rôles d'annuaire qui s'enchaînent vers Global Admin"
order: 1
description: "S'élever via les rôles d'annuaire Entra : abuser de Privileged Role Administrator, Application Administrator et de rôles similaires pour attribuer des rôles, réinitialiser des identifiants ou ajouter des secrets d'application."
keywords:
  - élévation de privilèges Entra
  - Privileged Role Administrator
  - Application Administrator
  - attribution de rôle
  - Global Administrator
---

# Élévation de privilèges

Plusieurs rôles Entra en deçà de Global Administrator l'atteignent malgré tout. **Privileged Role Administrator** peut attribuer n'importe quel rôle, y compris Global Admin, à n'importe qui. **Application / Cloud Application Administrator** peut ajouter des identifiants à n'importe quel service principal, puis agir comme une application qui détient des rôles Graph privilégiés. **Privileged Authentication Administrator** peut réinitialiser les identifiants et la MFA d'un Global Admin. Chacun est une chaîne en une étape vers le contrôle du tenant.

## S'attribuer un rôle

```bash
# Privileged Role Administrator: grant Global Administrator to a principal you hold
az rest --method POST \
  --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments" \
  --body '{"principalId":"<you>","roleDefinitionId":"62e90394-69f5-4237-9190-012177145e10","directoryScopeId":"/"}'
```

```bash
# Application Administrator: add a secret to a privileged app, then log in as it
az ad app credential reset --id <privileged-appId> --append
```

## Notes d'exploitation

- Le roleDefinitionId de Global Administrator est `62e90394-69f5-4237-9190-012177145e10` ; l'attribuer à la portée `/` est une prise de contrôle totale.
- Un Global Admin peut franchir vers Azure en basculant `elevateAccess` pour s'emparer de User Access Administrator sur chaque abonnement, faisant le pont vers le [plan de ressources Azure](../../../../online/cloud/azure/identity/privilege-escalation/elevate-access.md).
- Préférez la chaîne la plus discrète : ajouter un secret d'application (Application Administrator) se fond mieux dans la gestion normale des applications qu'une attribution de rôle directe.

## Outils

- **az cli** / **Graph** (`roleManagement/directory/roleAssignments`).
- **AzureHound** / **BARK** : trouver les arêtes de rôles qui atteignent Global Admin.

## Références

- [SpecterOps : BARK role escalation](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : Entra role privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Entra built-in roles](https://learn.microsoft.com/entra/identity/role-based-access-control/permissions-reference)
