---
title: "Introspection GraphQL activée en production : divulgation du schéma et des champs aux attaquants"
description: Serveurs GraphQL de production qui répondent aux requêtes d'introspection complètes du schéma et des types, cartographiant les resolvers et la forme des arguments pour les tests ultérieurs.
keywords:
  - introspection GraphQL
  - __schema
  - sécurité GraphQL
  - découverte d'API
---

# Introspection GraphQL

## Contexte

L'introspection (`__schema`, `__type`, listes de champs) est une fonctionnalité de première classe de GraphQL. Lorsqu'elle est exposée à des appelants de faible confiance dans un déploiement de production, elle devient une carte complète et lisible par machine des mutations, des types et des scalaires personnalisés. Cette page relève de la reconnaissance et de la cartographie de surface, elle ne remplace pas les tests d'autorisation par champ.

## Théorie

Le schéma montre quelles opérations existent, y compris les champs à consonance admin ou interne. Il accélère les tests BOLA et au niveau des champs en listant les noms d'arguments et les types de retour. Masquer l'introspection réduit la cartographie occasionnelle mais ne corrige pas une autorisation manquante sur les resolvers ; la valeur offensive est la rapidité et la complétude du graphe d'attaque dans le périmètre.

## Pratique

### Exécuter une requête d'introspection minimale en labo

- Faites un POST du corps de requête `__schema` standard vers l'endpoint HTTP GraphQL avec un jeton de test. Enregistrez le JSON de réponse pour une navigation hors ligne des types et des champs.

### Comparer le schéma au bundle client

- Comparez le schéma public aux requêtes du client mobile ou web. Les champs qui n'existent que dans l'introspection ou dans un build client interne sont prioritaires pour la recherche de contrôles manquants.

## Outils

- **GraphiQL** (en labo uniquement)
- **curl**
- **Insomnia**
