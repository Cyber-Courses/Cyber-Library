---
title: "Groupes role-assignable : l'appartenance accorde un rôle d'annuaire"
description: "Viser les groupes role-assignable : l'appartenance accorde les rôles d'annuaire liés au groupe, de sorte que le contrôle du groupe devient le contrôle du rôle."
keywords:
  - groupes role-assignable
  - isAssignableToRole
  - rôle d'annuaire
  - appartenance
  - élévation de privilèges
---

# Groupes role-assignable

Un groupe créé avec `isAssignableToRole` peut se voir attribuer des rôles d'annuaire, et chaque membre hérite de ces rôles. Ainsi, le contrôle d'un tel groupe, via la [propriété](group-ownership.md), l'[appartenance dynamique](dynamic-membership.md), ou un rôle capable de modifier ses membres, est le contrôle des rôles qui lui sont liés, potentiellement jusqu'à Global Administrator.

## Trouver et rejoindre

```bash
# list role-assignable groups
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups?\$filter=isAssignableToRole eq true"
# check which roles are assigned to the group, then get yourself added as a member
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments?\$filter=principalId eq '<group>'"
```

## Notes d'exploitation

- Les groupes role-assignable sont protégés (seuls des rôles plus privilégiés peuvent les gérer), mais un propriétaire défini à la création ou un gestionnaire mal cadré vous y fait quand même entrer.
- Un seul groupe role-assignable lié à Global Admin transforme toute arête d'appartenance en prise de contrôle du tenant.
- Ajouter un service principal contrôlé comme membre donne un chemin non interactif, exempt de MFA, vers le rôle.

## Outils

- **az cli** / **Graph** (`groups` isAssignableToRole, `roleAssignments`).
- **AzureHound** / **BARK** : arêtes groupe-vers-rôle.

## Références

- [SpecterOps : BARK group-to-role](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : role-assignable groups](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : role-assignable groups](https://learn.microsoft.com/entra/identity/role-based-access-control/groups-concept)
