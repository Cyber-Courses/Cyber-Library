---
title: "Mass assignment et overposting : lier des propriétés fournies par l'attaquant dans les API et les formulaires"
description: Lier du JSON client ou des champs de formulaire à des modèles de persistance sans liste d'autorisation, permettant de modifier des champs de rôle, de propriétaire ou de prix en une seule requête.
keywords:
  - mass assignment
  - overposting
  - liaison automatique
  - élévation de privilèges
---

# Mass assignment

## Contexte

Les frameworks mappent les corps de requête vers des entités d'ORM ou des modèles de documents. Si le binder copie chaque clé envoyée par le client, un seul `PATCH` peut définir `role`, `isAdmin`, `accountBalance` ou `ownerId` lorsque ces clés existent sur le modèle. Le signal est un champ sensible modifiable, pas un mauvais identifiant d'objet en lui-même (même si vous pouvez chaîner les deux).

## Théorie

Le code vulnérable utilise souvent un seul modèle ou DTO à la fois pour les mises à jour en libre-service à faibles privilèges et pour les champs à plus haut niveau de confiance de la même table. Les sémantiques de remplacement `PUT` et de fusion `PATCH` diffèrent : des clés imbriquées peuvent atterrir dans des propriétés profondes du graphe qu'un filtre de clés superficiel ne nomme jamais.

## Pratique

### Ajouter des clés de privilège à côté de champs valides en laboratoire

- Envoyez un petit corps JSON normalement autorisé (par exemple `displayName`) plus une seconde clé que l'interface n'affiche jamais, telle que `role` ou `isAdmin`, avec une valeur qui devrait être impossible pour le compte. Observez le code de réponse et un `GET` de suivi du même objet pour vérifier la persistance du champ.

## Outils

- **curl**
- **Burp Suite**
- **Postman**
