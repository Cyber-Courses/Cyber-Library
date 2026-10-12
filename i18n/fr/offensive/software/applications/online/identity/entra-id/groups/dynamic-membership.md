---
title: "Appartenance dynamique : correspondre à la règle d'un groupe pour le rejoindre"
description: "S'injecter dans un groupe de sécurité dynamique en définissant l'attribut utilisateur que sa règle d'appartenance évalue, héritant de l'accès et des rôles du groupe."
keywords:
  - dynamic membership
  - dynamic groups
  - membership rule
  - attribute injection
  - access
---

# Appartenance dynamique

Un groupe dynamique recalcule ses membres à partir d'une règle portant sur les attributs utilisateur (par exemple `user.department -eq "IT"`). Si vous pouvez modifier un attribut que la règle évalue, sur votre propre compte ou un compte que vous contrôlez, vous faites en sorte de correspondre et Entra vous ajoute automatiquement, héritant de tout ce que le groupe accorde.

## Correspondre à la règle

```bash
# read a dynamic group's membership rule
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups/<group>?\$select=membershipRule,groupTypes"

# set the matching attribute on a user you can edit (needs the right to write that attribute)
az rest --method PATCH --url "https://graph.microsoft.com/v1.0/users/<you>" \
  --body '{"department":"IT"}'
```

## Notes d'exploitation

- Les attributs modifiables en libre-service (intitulé de poste, service dans certains locataires) sont le point d'entrée ; les attributs inscriptibles varient selon le locataire et le rôle.
- L'appartenance se recalcule de façon asynchrone, il y a donc un court délai avant que l'accès ne soit effectif.
- Les groupes dynamiques sous-tendent souvent l'attribution d'applications et de licences, et parfois des groupes assignables à des rôles, de sorte que l'accès hérité peut être important.

## Outils

- **az cli** / **Graph** (`groups` membershipRule, `users` PATCH).
- **ROADtools** / **AzureHound** : règles et effets des groupes dynamiques.

## Références

- [HackTricks Cloud : groupes dynamiques](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps : AzureHound](https://github.com/BloodHoundAD/AzureHound)
- [Microsoft : règles d'appartenance dynamique](https://learn.microsoft.com/entra/identity/users/groups-dynamic-membership)
