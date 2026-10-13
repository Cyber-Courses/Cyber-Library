---
title: "Deviner des identifiants publics : IDs séquentiels, slugs courts et références publiques énumérables"
description: Slugs non listés mais devinables, jetons de liens courts et URLs de partage publiques qui renvoient des données sans contrôle d'habilitation dans le code.
keywords:
  - URL non listée
  - slug
  - lien court
  - BOLA
---

# Deviner les identifiants

## Contexte

« Toute personne disposant du lien » signifie souvent un long jeton non devinable. Lorsque le jeton est un slug anglais court ou une chaîne à faible entropie, la découverte relève de l'énumération, pas de la magie. Le serveur doit malgré tout appliquer côté serveur une règle sur qui peut utiliser un lien, et pas seulement s'appuyer sur l'obscurité.

## Théorie

Les moteurs de recherche, les référents (referrers) et les index publics peuvent déjà lister certaines pages « non listées ». La question offensive est de savoir ce qui se passe lorsqu'un second sujet demande le même slug avec une session différente ou sans session.

## Pratique

### Parcourir l'espace des slugs dans des programmes de test

- En laboratoire, générez des objets avec des slugs adjacents ou des codes publics numériques incrémentaux. Demandez les voisins pour voir si les slugs non possédés renvoient `404` de manière cohérente ou divulguent des métadonnées.

## Outils

- **ffuf**
- **curl**
- **Burp Suite**
