---
title: "Groupes Entra"
order: 4
description: "Prendre le contrôle des groupes de sécurité Entra pour hériter de leur accès : injection de règle d'appartenance dynamique, propriété de groupe, et groupes role-assignable qui accordent des rôles d'annuaire."
keywords:
  - groupes Entra
  - appartenance dynamique
  - propriété de groupe
  - groupes role-assignable
  - appartenance
---

# Groupes

Les groupes Entra contrôlent l'accès aux applications, aux ressources et même aux rôles d'annuaire, de sorte que devenir membre du bon groupe hérite de sa puissance sans toucher directement au rôle ou à la ressource. Les voies d'accès sont rejoindre un groupe dynamique en correspondant à sa règle, abuser de la propriété pour vous ajouter, et viser les groupes liés à des rôles d'annuaire.

## Ce qui est regroupé ici

- **[Appartenance dynamique](dynamic-membership.md)** : définir l'attribut auquel correspond la règle d'un groupe dynamique.
- **[Propriété de groupe](group-ownership.md)** : un propriétaire s'ajoutant comme membre.
- **[Groupes role-assignable](role-assignable-groups.md)** : groupes dont l'appartenance accorde un rôle d'annuaire.

## Références

- [SpecterOps : AzureHound group edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud : Entra groups](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Entra groups and membership](https://learn.microsoft.com/entra/identity/users/groups-create-rule)
