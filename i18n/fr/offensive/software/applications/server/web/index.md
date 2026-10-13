---
order: 10
title: "Surface d'attaque des applications web"
description: "Où vivent les vulnérabilités web : la plateforme qui héberge le service, le runtime qui l'exécute et le code applicatif écrit par les développeurs."
keywords:
  - web
  - sécurité des applications web
  - surface d'attaque
  - serveur
  - code applicatif
---

# Web

Un service web est attaqué sur trois couches : la **plateforme** (le processus serveur et la manière dont il est hébergé et câblé), le **runtime** (l'interpréteur du langage ou la machine virtuelle) et le **code** écrit par les développeurs. L'endroit où vit une vulnérabilité détermine comment on l'atteint et on la corrige : une faille dans la logique applicative se comporte très différemment d'une faille qui abuse du processus serveur ou de l'interpréteur.

Triez par couche : identifiez quelle couche possède la configuration, la primitive ou la logique défectueuse à partir des indices dont vous disposez (en-têtes, traces de pile, configuration, comportement), puis descendez dans le sous-arbre correspondant.

## Sous-sujets

- **[Code](code/index.md)** : failles qui proviennent du code source applicatif contrôlé par l'équipe de développement : composition non sûre de requêtes et de commandes, autorisation défaillante et logique métier...
- **[Plateforme](platform/index.md)** : techniques offensives contre la plateforme web organisées comme le déroule un audit : expositions générales valables pour tout serveur, puis mauvaises configurations par produit...
- **[Runtime](runtime/index.md)** : techniques offensives contre l'interpréteur ou la VM qui exécute le code des applications web : désérialisation non sûre, stream wrappers du moteur et échappement de l'assainissement du runtime...
