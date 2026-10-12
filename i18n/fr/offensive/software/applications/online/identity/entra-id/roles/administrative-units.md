---
title: "Unités administratives : abus des rôles cadrés"
order: 3
description: "Abuser des unités administratives : attributions de rôles cadrées et failles des UA à gestion restreinte qui délèguent le contrôle sur des sous-ensembles de l'annuaire."
keywords:
  - unités administratives
  - rôles cadrés
  - administration déléguée
  - gestion restreinte
  - UA
---

# Unités administratives

Une unité administrative cadre un rôle d'annuaire sur un sous-ensemble d'utilisateurs, de groupes ou d'appareils. Un **User Administrator** ou **Authentication Administrator** cadré sur une UA peut réinitialiser les mots de passe ou la MFA de chaque membre de cette UA, ce qui constitue une forte élévation si un utilisateur privilégié se trouve à l'intérieur, et une élévation discrète car le rôle paraît limité.

## Abuser d'un rôle cadré

```bash
# enumerate AUs and their scoped role assignments and members
az rest --method GET --url "https://graph.microsoft.com/v1.0/directory/administrativeUnits"
az rest --method GET --url "https://graph.microsoft.com/v1.0/directory/administrativeUnits/<au>/members"
# as a scoped User Administrator, reset a member's password / auth methods
```

## Notes d'exploitation

- Un Authentication Administrator cadré sur une UA qui contient un utilisateur privilégié peut réinitialiser les identifiants de cet utilisateur, s'élevant au-delà de la limite apparente de l'UA.
- Les UA à gestion restreinte sont censées protéger les membres, mais un cadrage mal configuré et des attributions qui se chevauchent créent des failles qu'il vaut la peine d'énumérer.
- Les rôles cadrés paraissent à faible risque dans une revue de rôles, ce qui en fait un bon point d'ancrage discret.

## Outils

- **az cli** / **Graph** (`directory/administrativeUnits`).
- **ROADtools** / **AzureHound** : graphe des UA et des attributions cadrées.

## Références

- [HackTricks Cloud : administrative units](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : AzureHound](https://github.com/BloodHoundAD/AzureHound)
- [Microsoft : administrative units](https://learn.microsoft.com/entra/identity/role-based-access-control/administrative-units)
