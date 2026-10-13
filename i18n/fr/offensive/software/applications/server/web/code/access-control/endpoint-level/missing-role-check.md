---
title: "Contrôles de rôle et de permission manquants sur les handlers HTTP, les contrôleurs et les resolvers GraphQL"
description: Handlers qui authentifient l'appelant mais omettent les contrôles de permission au niveau applicatif pour la route, la mutation ou le champ GraphQL spécifique.
keywords:
  - autorisation manquante
  - autorisation au niveau fonction
  - élévation de privilèges verticale
  - contrôle de rôle
  - contrôle d'accès défaillant
---

# Contrôles de rôle manquants

## Contexte

Le sujet est déjà authentifié ; la défaillance est que le chemin de code exécute une opération sensible (rapport admin, changement d'état, export) sans faire correspondre le rôle, la portée ou le tenant du sujet à l'opération. Le BOLA voisin au niveau objet est une classe différente : la route est « bonne » par son nom, mais l'identifiant d'objet est mauvais. Cette page porte sur les lacunes au niveau **fonction**.

## Théorie

Formes courantes : une nouvelle route est livrée sans la garde que l'ancien schéma utilisait ; un resolver de champ GraphQL omet un contrôle au niveau du champ que le niveau du type suppose ; un microservice fait confiance au réseau interne et saute l'autorisation d'entrée ; un chemin versionné duplique un comportement avec une matrice de politique plus faible. Fuzzez par matrice de rôles (comptes à faibles et à forts privilèges) et par méthode HTTP et type de contenu sur le même chemin.

## Pratique

### Cartographier la matrice route/rôle en labo

- Avec deux comptes de test de rôles différents, appelez le même chemin et la même méthode avec la même forme de corps. Si une réponse est `403` pour le premier compte et `200` pour le second sur une fonction réservée aux admins, la politique existe ; si les deux renvoient `200`, le contrôle au niveau fonction est probablement manquant pour cet ensemble de routes.

### Comparer l'application au niveau passerelle et au niveau applicatif

- Lorsqu'une passerelle ou un BFF est en jeu, demandez la même route interne à la fois via la passerelle et (en labo de dev uniquement) directement sur un port applicatif s'il est exposé. Un contrôle manquant n'apparaît souvent que sur un seul chemin.

### Surface des champs GraphQL

- Listez les noms de champs d'un type dans un schéma de test ou un bundle de documentation, puis appelez chaque champ avec un jeton à faibles privilèges. Les contrôles manquants au niveau du champ se manifestent souvent par des `data` pour des champs qui devraient être des `errors` avec une extension d'authentification.

## Outils

- **curl**
- **Postman**
- **Burp Suite**
