---
title: "Actifs et surface d'attaque : ce que vous protégez, et par où on peut l'atteindre"
description: "Les actifs sont les éléments qui méritent d'être protégés, et la surface d'attaque est l'ensemble des points par lesquels ces actifs peuvent être atteints ou attaqués."
keywords:
  - actifs
  - surface d'attaque
  - réduction de la surface d'attaque
  - inventaire des actifs
  - exposition
  - fondamentaux de la sécurité
---

# Actifs et surface d'attaque

**Un actif est tout élément de valeur qu'une organisation doit protéger, et la surface d'attaque est l'ensemble des points par lesquels ces actifs peuvent être atteints, sondés ou attaqués.** Les deux notions vont de pair : vous ne pouvez pas défendre ce que vous n'avez pas identifié comme digne d'être défendu, et vous ne pouvez pas mesurer votre exposition tant que vous ne savez pas par où un attaquant peut entrer. Le risque n'existe que là où un actif de valeur rencontre un moyen de l'atteindre, c'est pourquoi presque toute décision de sécurité commence par ces deux questions.

## Déterminer ce qui constitue un actif

Un actif ne se limite pas aux données. Le travail de sécurité regroupe généralement les actifs en quelques catégories, et chacune se dégrade à sa manière lorsqu'elle est perdue ou détournée :

| Type d'actif | Exemples | Ce que signifie sa perte |
| --- | --- | --- |
| Information | Dossiers clients, code source, identifiants, données financières, propriété intellectuelle | Divulguée, altérée ou prise en otage contre rançon |
| Systèmes et logiciels | Serveurs, ordinateurs portables, appareils mobiles, applications et les services qui les font tourner | Pris en main, perturbés ou utilisés comme point d'appui |
| Infrastructure et comptes | Réseaux, locataires cloud, domaines et les identités qui les contrôlent | Prise de contrôle de tout ce qui repose dessus |
| Personnes et réputation | Le savoir du personnel, et la confiance dont une organisation dépend pour fonctionner | Personnes internes manipulées, clients et partenaires perdus |

Le critère utile est la valeur : si sa confidentialité, son intégrité ou sa disponibilité compte pour l'organisation, c'est un actif (voir la [triade CIA](cia-triad.md)). Les identifiants et les identités méritent une attention particulière, car en compromettre un ouvre souvent un chemin vers de nombreux autres actifs à la fois.

## Cartographier par où chaque actif peut être atteint

La surface d'attaque est chaque endroit où l'un de ces actifs est exposé à une interaction. Le NIST la définit comme l'ensemble des points situés à la frontière d'un système ou d'un environnement par lesquels un attaquant peut tenter d'entrer, de provoquer un effet ou d'extraire des données. En pratique, ces points se répartissent en quelques familles :

- **Réseau :** chaque port ouvert, service exposé et passerelle d'accès à distance.
- **Logiciel :** pages de connexion, points de terminaison d'API, formulaires de téléversement de fichiers et toute entrée qu'une application accepte.
- **Physique :** ports USB, postes de travail laissés sans surveillance et appareils qui peuvent être emportés.
- **Tiers :** intégrations, fournisseurs et comptes de prestataires qui disposent d'un chemin vers votre environnement.
- **Humain :** chaque personne que l'on peut appeler, contacter par e-mail ou hameçonner (phishing), ce que l'on appelle parfois la surface d'attaque humaine. Même une adresse e-mail du personnel publiée est un point de la surface.

## Une base de données, trois expositions

Prenons une seule base de données clients, et voyons comment son exposition change selon ce qui l'entoure :

1. **Isolée sur un réseau interne sans aucun chemin entrant.** La surface est réduite : un attaquant doit d'abord pénétrer dans le réseau, et seuls quelques administrateurs peuvent atteindre la base.
2. **Derrière un formulaire web public.** Les mêmes données sont désormais accessibles à quiconque sur Internet peut soumettre une saisie, si bien que chaque faille du formulaire et de l'application qui se trouve derrière fait partie de la surface.
3. **Également synchronisée avec la plateforme d'analyse d'un prestataire.** L'actif a désormais un second emplacement, et le personnel, les comptes et les pratiques de sécurité du prestataire rejoignent la surface, alors que rien n'a changé sur vos propres serveurs.

Les données sont les mêmes dans les trois cas ; seule leur accessibilité a changé. Une surface plus grande signifie davantage de points qu'un défenseur doit corriger, surveiller et analyser, et davantage de points qu'un attaquant peut étudier à la recherche de [vulnérabilités](vulnerabilities-and-exploits.md).

## Réduire la surface sans entraver l'activité

Réduire la surface diminue directement le risque : fermer un port inutilisé ou retirer un serveur oublié écarte toute une catégorie d'attaques sans qu'il soit nécessaire de les détecter. Les équipes procèdent généralement dans cet ordre :

1. **Inventorier ce qui existe.** Cartographier les actifs et ce à quoi ils servent, car les actifs inconnus ne sont défendus que par accident, quand ils le sont.
2. **Supprimer ce qui n'est pas nécessaire.** Fermer les services inutilisés, mettre hors service les anciens systèmes et supprimer les comptes et intégrations obsolètes.
3. **Restreindre l'accès à ce qui reste.** Limiter qui et quoi peut se connecter à chaque actif, en appliquant le [moindre privilège](../principles/least-privilege.md) aux accès et en réduisant la collecte de données au [strict minimum](../principles/minimization.md).
4. **Protéger les points qui doivent rester ouverts.** Placer des [contrôles et contre-mesures](controls-and-countermeasures.md) sur les points exposés dont l'activité a réellement besoin.

## Les actifs que tout le monde a oubliés

Le plus difficile n'est généralement pas de défendre les actifs connus, mais de trouver ceux dont plus personne ne se souvenait : un serveur de test resté en ligne, un espace de stockage cloud créé pour un projet ponctuel, un outil SaaS adopté par une équipe à l'insu du service informatique (shadow IT). Comme les ressources cloud et les nouvelles intégrations apparaissent plus vite qu'un audit ponctuel ne peut suivre, la gestion de la surface d'attaque est désormais une discipline continue de découverte plutôt qu'une liste de contrôle annuelle. Les [acteurs de la menace](threats-and-threat-actors.md) recherchent précisément ces points oubliés, si bien qu'une organisation qui n'examine pas sa propre surface depuis l'extérieur laisse ce point de vue à ses attaquants.

## Références

- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [NIST Glossary: Asset](https://csrc.nist.gov/glossary/term/asset)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CIS Control 1: Inventory and Control of Enterprise Assets](https://www.cisecurity.org/controls/inventory-and-control-of-enterprise-assets)
- [CISA: Foundations for OT Cybersecurity, Asset Inventory Guidance for Owners and Operators](https://www.cisa.gov/resources-tools/resources/foundations-ot-cybersecurity-asset-inventory-guidance-owners-and-operators)
