---
title: "Actifs et surface d'attaque : ce que la sécurité protège et par où on peut l'atteindre"
description: "Ce que la sécurité protège, et l'ensemble de tous les points par lesquels ces actifs peuvent être atteints ou attaqués."
keywords:
  - actifs
  - surface d'attaque
  - réduction de la surface d'attaque
  - inventaire des actifs
  - exposition
  - accessibilité
---

# Actifs et surface d'attaque

**Un actif est tout ce dont la perte nuirait à une organisation, et la surface d'attaque est l'ensemble complet des points par lesquels ces actifs peuvent être atteints ou attaqués.** Les deux notions n'ont de sens qu'ensemble. Un actif sans aucune voie d'accès est hors de portée d'un attaquant, même si un accident, une panne ou un incendie peut encore lui nuire, et une voie d'accès qui ne mène à rien de précieux ne vaut pas le temps d'un attaquant. La sécurité réside dans le recoupement : une chose de valeur, plus un chemin qui y mène.

## La liste du défenseur face à la carte de l'attaquant

Un défenseur a tendance à raisonner en termes de liste : les données, les systèmes et les comptes dont l'entreprise dépend. Un attaquant raisonne en termes de carte : chaque endroit où cette liste peut être touchée depuis l'extérieur. L'écart entre les deux est l'endroit où surviennent les incidents. Tout ce qui figure sur la liste du défenseur mais pas sur la carte de l'attaquant est hors d'atteinte en toute sécurité ; tout ce qui figure sur la carte de l'attaquant mais pas sur la liste du défenseur est une exposition que personne ne surveille. L'objectif de ce couplage est de faire coïncider les deux vues, afin que chaque actif de valeur soit pris en compte et que chaque point accessible soit connu.

## Quatre types d'actifs, quatre façons de les perdre

Un actif est rarement un simple fichier. Le travail de sécurité les regroupe par type, car chaque type est mis en défaut différemment lorsque sa [confidentialité, son intégrité ou sa disponibilité](cia-triad.md) est compromise.

| Type | Exemples | Ce à quoi ressemble sa perte |
| --- | --- | --- |
| Information | Dossiers, code source, secrets | Fuite, altération ou prise en otage |
| Systèmes | Serveurs, ordinateurs portables, applications | Détournement ou interruption |
| Identités | Comptes, clés, domaines | Contrôle remis à un attaquant |
| Confiance | Savoir du personnel, réputation | Personnel induit en erreur, clients perdus |

Les identités méritent le plus grand soin, car un seul identifiant volé s'arrête rarement à un seul actif.

## La frontière où commencent les attaques

Le NIST définit la surface d'attaque comme l'ensemble des points situés à la frontière d'un système par lesquels un attaquant peut tenter d'entrer, de produire un effet ou d'extraire des données. En pratique, ces points de frontière se répartissent en quelques familles :

- **Réseau :** ports ouverts, services exposés, passerelles d'accès à distance.
- **Logiciel :** pages de connexion, points de terminaison d'API, formulaires de téléversement, toute entrée acceptée.
- **Physique :** ports USB, machines laissées sans surveillance, appareils portables.
- **Humain :** toute personne susceptible d'être victime de phishing ou d'un appel frauduleux.
- **Chaîne d'approvisionnement :** fournisseurs et intégrations qui détiennent un chemin vers l'intérieur.

Chaque point est un endroit où une [vulnérabilité](vulnerabilities-and-exploits.md) pourrait se nicher et être découverte.

## Ajouter une fonctionnalité, élargir la surface

Observez comment l'exposition d'un seul actif évolue à mesure que l'entreprise se développe autour de lui. La base de données clients ne bouge jamais, mais son accessibilité, elle, change.

1. **En interne uniquement.** La base de données se trouve sur un réseau privé. La surface se limite à une poignée d'administrateurs.
2. **Un formulaire web public est ajouté.** N'importe qui sur Internet peut désormais soumettre des données, de sorte que chaque faille de ce formulaire rejoint la surface.
3. **L'analyse de données est externalisée.** Une copie est synchronisée vers un fournisseur. Le personnel, les comptes et la sécurité du fournisseur se trouvent désormais entre un attaquant et vos données, sans rien avoir changé sur vos propres serveurs.

Les données sont identiques dans les trois étapes. Seul le nombre de chemins qui y mènent a augmenté.

## Les surfaces croissent plus vite que les inventaires

L'actif le plus difficile à défendre est celui dont personne ne se souvient : un serveur de test laissé en ligne, un espace de stockage issu d'un ancien projet, un outil adopté par une équipe sans prévenir personne. Les ressources cloud apparaissent plus vite que n'importe quel audit annuel, c'est pourquoi la gestion de la surface d'attaque est aujourd'hui une découverte continue plutôt qu'une liste de contrôle. Réduire la surface, en fermant les services inutilisés et en appliquant le [moindre privilège](../principles/least-privilege.md) à ce qui reste, retire des catégories entières d'attaques avant même qu'un contrôle de détection ne soit nécessaire. Les [acteurs de la menace](threats-and-threat-actors.md) recherchent en permanence les points oubliés, de sorte qu'une organisation qui ne regarde jamais sa propre surface depuis l'extérieur leur en laisse entièrement la vue.

## Références

- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [NIST Glossary: Asset](https://csrc.nist.gov/glossary/term/asset)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CIS Control 1: Inventory and Control of Enterprise Assets](https://www.cisecurity.org/controls/inventory-and-control-of-enterprise-assets)
