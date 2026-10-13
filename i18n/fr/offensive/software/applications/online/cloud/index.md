---
title: "Cloud"
order: 1
description: "Techniques offensives contre les plateformes cloud, organisées par fournisseur car le modèle d'identité, les services, les API et l'outillage de chacun diffèrent. Un compte cloud est constitué d'identités détenant des permissions sur des ressources, accessibles via des identifiants, de sorte que les attaques tournent autour du graphe IAM et de l'API du plan de contrôle plutôt que des exploits d'hôte."
keywords:
  - sécurité du cloud
  - AWS
  - Azure
  - GCP
  - IAM
---

# Cloud

Les plateformes cloud sont attaquées via leurs **API de plan de contrôle**, et non par l'exploitation d'hôtes. Un compte cloud est un ensemble d'**identités** détenant des **permissions** sur des **ressources**, accessibles avec des **identifiants** ; le travail consiste donc à énumérer ce graphe, à obtenir un principal, à élever ses permissions, puis à abuser des services qu'il peut atteindre. Comme le modèle d'identité, l'ensemble de services et l'outillage de chaque fournisseur diffèrent fortement, ce domaine est organisé **par fournisseur**, et chaque fournisseur est découpé de la même manière, par la **surface** attaquée plutôt que par phase de la chaîne d'attaque.

## Comment chaque fournisseur est organisé

Les neuf mêmes surfaces reviennent chez chaque fournisseur, de sorte qu'une technique se trouve à côté de la ressource ou du mécanisme qu'elle exploite :

- **Identité** et **Identifiants** constituent le cœur du plan de contrôle : obtenir un principal, puis l'élever.
- **Compute**, **Stockage**, **Sans serveur**, **Données**, **Réseau** et **Messagerie** sont les surfaces de ressources.
- **Journalisation et détection** est le plan d'audit, attaqué pour l'aveugler et l'éviter.

L'énumération, l'élévation de privilèges, le mouvement latéral, la persistance et l'exfiltration ne sont pas des sujets séparés : chacun se trouve dans la surface par laquelle il opère. L'énumération est intégrée à chaque surface (cartographier le graphe IAM sous l'identité, trouver des buckets publics sous le stockage) ; l'élévation de privilèges se trouve sous l'identité, l'exfiltration sous le stockage et les données, et ainsi de suite. L'outillage d'inventaire à l'échelle du compte qui traverse les surfaces est couvert en tant que méthodologie dans l'index de chaque fournisseur.

## Fournisseurs

- **[AWS](aws/index.md)** : Amazon Web Services, le modèle de référence pour le découpage par surface ci-dessus.
- **[Azure](azure/index.md)** : le plan de ressources et de gestion Microsoft Azure (ARM, RBAC, identités managées).
- **[GCP](gcp/index.md)** : Google Cloud Platform, piloté via Cloud IAM, les comptes de service, et les chemins de déploiement `actAs`.

## Portée et limites

- **Entra ID** (l'annuaire cloud de Microsoft) est attaqué comme un annuaire, il se trouve donc sous [Entra ID](../identity/entra-id/index.md) à côté d'Active Directory, et non ici ; le domaine Azure ici couvre le **plan de ressources** (abonnements, VM, stockage, identités managées).
- **Kubernetes et les évasions de conteneurs** constituent leur propre domaine ; les pages cloud ne couvrent que le plan de contrôle des services managés et renvoient vers l'interne des conteneurs.
- Le pont entre l'environnement sur site et le cloud (Entra Connect, AD FS, PRT) se trouve sous [identité hybride](../../server/directory/active-directory/trusts/entra-hybrid.md).

## Références

- [HackTricks Cloud](https://cloud.hacktricks.wiki/en/index.html)
- [MITRE ATT&CK: cloud matrices](https://attack.mitre.org/matrices/enterprise/cloud/)
