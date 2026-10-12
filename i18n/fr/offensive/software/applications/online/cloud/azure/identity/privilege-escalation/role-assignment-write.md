---
title: "Écriture d'attribution de rôle : se donner Owner"
order: 1
description: "S'attribuer, ou attribuer à un principal contrôlé, Owner ou Contributor avec Microsoft.Authorization/roleAssignments/write au scope ressource, groupe, abonnement ou groupe d'administration."
keywords:
  - roleAssignments write
  - Owner
  - Contributor
  - Azure RBAC
  - scope
  - Microsoft.Authorization
---

# Écriture d'attribution de rôle

`Microsoft.Authorization/roleAssignments/write` est la permission d'accorder des rôles RBAC. Un principal qui la détient (via Owner, User Access Administrator ou un rôle personnalisé) peut s'attribuer, ou attribuer à un principal qu'il contrôle, un rôle supérieur à n'importe quel scope qu'il atteint : une ressource unique, un groupe de ressources, un abonnement ou un groupe d'administration entier.

## Accorder Owner

```bash
# subscription-wide Owner to yourself
az role assignment create --assignee <your-object-id> \
  --role Owner --scope /subscriptions/<sub-id>

# or scope it to a resource group / single resource
az role assignment create --assignee <object-id> --role Contributor \
  --scope /subscriptions/<sub>/resourceGroups/<rg>
```

Un principal de service que vous contrôlez est un bénéficiaire plus discret que votre propre utilisateur, et survit à la désactivation de l'utilisateur.

## Notes d'exploitation

- Le scope le plus élevé sur lequel vous pouvez écrire est ce qui compte : `roleAssignments/write` au niveau d'un groupe d'administration accorde jusqu'à chaque abonnement situé en dessous.
- Owner inclut `roleAssignments/write` ; Contributor non, donc Contributor ne peut pas se donner Owner. Cette frontière explique pourquoi [elevate-access](elevate-access.md) et les [rôles personnalisés](custom-role-definition.md) comptent quand vous ne détenez que des droits de type Contributor.
- Les attributions sont visibles dans l'Activity Log et le portail ; une attribution restreinte sur un groupe de ressources obscur attire moins l'attention qu'une attribution Owner à l'échelle de l'abonnement.

## Outils

- **az cli** (`az role assignment create`).
- **MicroBurst** (les modules de l'époque `Invoke-AzureRmWebAppsExfil` énumèrent et abusent du RBAC).
- **BARK** (`New-AzureRMRoleAssignment`) : attribution scriptée depuis un token.

## Références

- [SpecterOps : abuser d'Azure RBAC avec BARK](https://github.com/BloodHoundAD/BARK)
- [Microsoft : rôles intégrés Azure](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles)
- [HackTricks Cloud : privesc Azure RBAC](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
