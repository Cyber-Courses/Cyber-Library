---
title: "Approbations inter-forêts : se déplacer entre forêts"
order: 3
description: "Se déplacer à travers une approbation de forêt Active Directory, où le filtrage de SID et l'authentification sélective sont appliqués par défaut : utiliser les accès explicitement accordés aux principaux étrangers, les approbations au filtrage de SID assoupli, et la délégation de TGT vers des hôtes à délégation sans contrainte à travers l'approbation."
keywords:
  - forest trust
  - SID filtering
  - selective authentication
  - TGT delegation
  - foreign security principal
---

# Approbations inter-forêts

Une approbation de forêt relie deux forêts distinctes et, contrairement à une approbation intra-forêt, elle est censée constituer une véritable frontière. Par défaut, elle applique le **filtrage de SID** (quarantaine), qui retire les SID n'appartenant pas à la forêt approuvée, de sorte que l'astuce du SID history qui traverse les domaines à l'intérieur d'une forêt ne **traverse pas** une approbation de forêt. Le déplacement à travers une approbation de forêt est donc plus étroit et dépend de ce que l'approbation et ses administrateurs autorisent réellement.

## Ce que font les garde-fous

- **Filtrage de SID / quarantaine** : appliqué par défaut sur une approbation de forêt, il retire les SID étrangers (y compris `extraSids`) des tickets entrants, de sorte qu'un SID privilégié forgé dans la forêt A est retiré avant que la forêt B ne l'honore.
- **Authentification sélective** : un réglage **à activer** (`/selectiveauth:Yes`), pas une valeur par défaut. La valeur par défaut est l'**authentification à l'échelle de la forêt**, où les principaux de la forêt approuvée peuvent s'authentifier et où l'accès est régi par les ACL de ressources ordinaires. Avec l'authentification sélective activée, les principaux étrangers n'obtiennent rien à moins que « Autorisé à s'authentifier » ne leur soit explicitement accordé sur chaque ressource.
- **Direction et transitivité de l'approbation** : une approbation de forêt peut être unidirectionnelle et est non transitive vers des forêts tierces, ce qui limite la portée.

## Chemins qui fonctionnent encore

- **Accès accordé aux principaux étrangers** : les organisations ajoutent des utilisateurs ou des groupes de l'autre forêt à des groupes locaux et à des ACL (foreign security principals). Ces octrois sont légitimes et non filtrés, énumérez donc ce que les principaux de la forêt approuvée peuvent déjà atteindre et utilisez-le.
- **Filtrage de SID assoupli** : le filtrage de SID peut être désactivé ou relâché sur une approbation (par exemple pour prendre en charge des migrations), rouvrant l'injection de SID history à travers l'approbation. Lisez la posture de filtrage de l'approbation lors de l'[énumération des approbations](trust-enumeration.md) ; là où il est désactivé, traitez l'approbation de forêt comme une approbation intra-forêt.
- **Délégation de TGT vers la délégation sans contrainte** : si la délégation de TGT est activée sur l'approbation et que la forêt approuvée possède un hôte en [délégation sans contrainte](../authentication/kerberos/delegation/unconstrained.md), contraindre un compte privilégié à travers l'approbation à s'authentifier auprès de cet hôte capture son TGT, reliant les forêts.

## Notes d'exploitation

- Partez de ce qui est **déjà accordé** : la compromission inter-forêts vient généralement d'une ACL ou d'une appartenance de groupe mal configurée qui enjambe l'approbation, pas du contournement du filtrage de SID.
- Le filtrage de SID **activé** est la valeur par défaut et la première chose à vérifier ; une approbation marquée avec le SID history activé ou la quarantaine désactivée réduit la frontière à une difficulté intra-forêt.
- L'authentification sélective, là où elle est activée, signifie que même des identifiants étrangers valides n'obtiennent rien sans un octroi par ressource, cherchez donc spécifiquement ces octrois.

## Outils

- **BloodHound** : cartographie les ACL inter-forêts, les appartenances à des groupes étrangers et les principaux atteignables.
- **Impacket / Rubeus** : demander et utiliser des tickets inter-forêts une fois qu'un chemin existe.
- **nltest / Get-ADTrust** : lire la direction, la transitivité et les attributs de filtrage de l'approbation.

## Références

- The Hacker Recipes: cross-forest trusts
- Microsoft: SID filtering, quarantine, and selective authentication
