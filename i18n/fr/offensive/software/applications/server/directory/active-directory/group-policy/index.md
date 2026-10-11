---
title: "Group Policy : exécution de code et configuration à travers le domaine"
order: 3
description: "La stratégie de groupe Active Directory comme surface d'attaque : éditer une GPO que vous pouvez écrire, en lier une à une OU que vous contrôlez, et récupérer les identifiants laissés dans SYSVOL, pour exécuter du code sur chaque ordinateur et utilisateur auquel la stratégie s'applique."
keywords:
  - stratégie de groupe
  - abus de GPO
  - SYSVOL
  - gPLink
  - tâche planifiée
---

# Group Policy

La stratégie de groupe applique de la configuration et des scripts depuis le domaine vers les ordinateurs et utilisateurs dans sa portée. Un objet de stratégie de groupe (GPO) est un objet d'annuaire plus un dossier dans SYSVOL, et il est appliqué automatiquement par chaque machine à laquelle il est lié. Cela fait d'une GPO un mécanisme de livraison de code et de configuration à travers de nombreux hôtes à la fois : **un accès en écriture à une GPO, ou la capacité d'en lier une à une unité organisationnelle, est une exécution de code à distance sur tout ce qui est dans la portée.**

## Pourquoi les GPO sont une cible puissante

- Une seule GPO liée peut toucher **chaque ordinateur d'une OU ou du domaine entier**, donc une écriture élève largement en une seule étape.
- Les GPO peuvent déployer des **tâches planifiées, des scripts de démarrage/d'ouverture de session, des services et des changements de registre**, qui sont tous des primitives directes d'exécution ou de privilège.
- Le droit de **lier** une GPO à une OU (l'attribut `gPLink`) est aussi puissant que d'en éditer une : liez une GPO contrôlée par l'attaquant à une OU cible et elle s'applique.
- L'application est automatique selon le cycle de rafraîchissement de la stratégie, l'abus ne nécessite donc aucune interaction supplémentaire.

## Comment c'est exploité

- **Éditer une GPO accessible en écriture** pour ajouter une tâche planifiée immédiate ou un script qui s'exécute en tant que SYSTEM sur les machines dans la portée.
- **Lier une GPO malveillante** à une OU dont vous pouvez écrire le `gPLink`, amenant les objets dans la portée sous son contrôle.
- **Viser la bonne portée** : une GPO liée à l'OU Domain Controllers, ou à une OU tier-0, convertit le contrôle de la GPO en compromission du domaine.

## Pages

- **[Énumération des GPO et des OU](gpo-and-ou-enumeration.md)** : trouver les GPO accessibles en écriture, leurs liaisons et les OU que vous pouvez affecter.
- **[Éditer une GPO](editing-a-gpo.md)** : tâches planifiées immédiates, scripts et admin local sur tout ce qui est dans la portée.
- **[Lier une GPO](linking-a-gpo.md)** : `WriteGPLink` sur une OU pour amener ses objets dans une stratégie.
- **[Group Policy Preferences](group-policy-preferences.md)** : récupérer les identifiants `cpassword` et d'autologon depuis SYSVOL.

## Outils

- **pyGPOAbuse** (Linux) / **SharpGPOAbuse** (Windows) : injecter des tâches immédiates, des scripts et des changements d'admin local dans une GPO accessible en écriture.
- **NetExec (`nxc`)** : `ldap --gpo` pour lister les GPO, `-M gpp_password` / `-M gpp_autologin` pour les identifiants SYSVOL.
- **PowerView** (`New-GPOImmediateTask`, `Set-DomainObject` pour `gPLink`), **Group3r / Grouper2** : éditions de GPO et audits de paramètres vulnérables.
- **bloodyAD** : définir le `gPLink` d'une OU depuis Linux.

## Références

- The Hacker Recipes: Group policies
- Microsoft: Group Policy and SYSVOL
