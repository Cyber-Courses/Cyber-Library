---
title: "Active Directory : attaquer le domaine Windows et son écosystème d'authentification"
description: "La surface d'attaque Active Directory organisée selon ce qui est attaqué : reconnaissance du domaine, authentification et identifiants (NTLM, Kerberos, certificats), DACL, stratégie de groupe, approbations et comptes gérés."
keywords:
  - active directory
  - kerberos
  - ntlm
  - DACL
  - domain compromise
  - AD CS
---

# Active Directory

Active Directory (AD) est la colonne vertébrale de l'identité et de l'autorisation de la plupart des entreprises Windows : une base de données répliquée et accessible par LDAP d'utilisateurs, d'ordinateurs, de groupes et de stratégies, authentifiée avec Kerberos et NTLM et, de plus en plus, avec des certificats. Un attaquant n'a presque jamais besoin d'une vulnérabilité logicielle pour en prendre le contrôle. Le domaine est compromis en **abusant de ses fonctionnalités prévues** : attributs d'objets lisibles, permissions délégables, tickets de service déchiffrables, authentification relayable et entrées de contrôle d'accès trop permissives.

## Comment ce domaine est organisé

Ce domaine est organisé par **surface d'attaque**, la chose dont on abuse, car c'est la primitive durable et la façon dont une technique se recherche : le Kerberoasting est une technique Kerberos quelle que soit la phase de l'engagement où vous l'utilisez. Chaque sujet couvre son propre cycle de vie, de sa découverte à son abus puis à la persistance à travers lui, plutôt que de scinder tout cela en sections distinctes de chaîne d'attaque. La **reconnaissance** générale, lire l'annuaire pour trouver quoi attaquer, vient en premier comme surface à part entière. La plus grande surface de loin est ensuite l'**authentification**, puisque les mots de passe, les empreintes NTLM, les tickets Kerberos et les certificats sont tous des formes d'une même chose : du matériel d'authentification que vous obtenez et réutilisez. L'escalade fondée sur les permissions (DACL, stratégie de groupe), l'abus d'approbations au-delà des frontières et les comptes qu'AD gère pour vous sont chacun leur propre surface.

## L'arc de l'attaque

Un chemin typique traverse ces surfaces dans l'ordre :

1. **Reconnaissance** : lire le domaine depuis n'importe quelle position authentifiée (souvent n'importe quelle position réseau) pour cartographier utilisateurs, groupes, ordinateurs, délégations, ACL et approbations.
2. **Obtenir du matériel d'authentification** : pulvériser des mots de passe ou faire du roasting pour obtenir des secrets déchiffrables, extraire des identifiants depuis un point d'ancrage, relayer une authentification contrainte, ou abuser de modèles de certificats.
3. **Escalader** en abusant des permissions d'objets (DACL) et de la stratégie de groupe qui permettent à un principal contrôlé de réécrire des objets privilégiés.
4. **Franchir les frontières** en abusant des approbations de domaine et de forêt.
5. **Persister** avec des tickets forgés, l'abus de réplication et des portes dérobées sur des objets privilégiés, traité au sein de chaque surface à laquelle cela appartient.

## Sujets

- **[Authentification](authentication/index.md)** : extraction et cassage d'identifiants, NTLM, Kerberos et certificats (AD CS).
- **[DACL](dacl/index.md)** : abuser des permissions d'objets pour contrôler des principaux privilégiés.
- **[Stratégie de groupe](group-policy/index.md)** : abuser des GPO pour exécuter du code et modifier la configuration.
- **[Approbations](trusts/index.md)** : abus des approbations intra-forêt et inter-forêts.
- **[Comptes gérés](managed-accounts/index.md)** : secrets gMSA et LAPS dans l'annuaire, et escalade dMSA (BadSuccessor).

## Références

- The Hacker Recipes: Active Directory
- Microsoft: Active Directory Domain Services documentation
