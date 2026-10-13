---
order: 2
title: "Injection de filtre Prisma"
description: "Étaler un objet non fiable dans un argument where de Prisma laisse un appelant injecter des opérateurs et des filtres de relation, élargissant ou contournant la requête prévue sans aucun SQL brut."
keywords:
  - Prisma where
  - injection de filtre
  - injection d'opérateur
  - filtre de relation
  - étalement d'objet
---

# Injection de filtre

L'API typée de Prisma est sûre pour les valeurs, mais elle fait confiance à la forme des objets qu'on lui donne. Un argument `where` est un objet imbriqué dont les clés sont des champs et des opérateurs, et un endpoint qui étale un corps de requête non fiable dedans remet à l'appelant le contrôle de la structure de la requête, pas seulement d'une valeur de filtre.

## Étalement de l'entrée de requête dans where

```js
// UNSAFE: client controls the shape of the filter
const users = await prisma.user.findMany({
  where: { ...req.body.filter },
});
```

Si l'endpoint attendait `{ email: "a@b.c" }`, un appelant peut envoyer des opérateurs à la place :

```json
{ "filter": { "email": { "not": "" } } }
{ "filter": { "role": { "equals": "ADMIN" } } }
{ "filter": { "OR": [ { "id": 1 }, { "id": 2 } ] } }
```

`{ "not": "" }` correspond à presque toutes les lignes, `OR` élargit l'ensemble, et une clé que l'endpoint n'avait jamais prévu de rendre filtrable (`role`, `isAdmin`) devient un sélecteur. Lorsque `where` s'étend à travers les relations, un filtre de relation imbriqué fait pivoter la requête vers des enregistrements liés que l'appelant ne devrait pas pouvoir contraindre.

## Conversion de type dans un filtre

Passer une valeur là où Prisma attend un scalaire peut aussi changer le comportement : un champ typé comme chaîne qui reçoit un objet (`{"email": {"startsWith": "admin"}}`) est interprété comme un filtre d'opérateur plutôt qu'une égalité, donc un endpoint qui fait `where: { email: req.query.email }` sans valider le type laisse l'appelant passer de l'égalité à une correspondance par préfixe ou par négation.

## Confirmer la faille

Envoyez des objets opérateur là où l'endpoint attend un scalaire et comparez le jeu de résultats : un corps `{ "not": null }`, `{ "gt": 0 }`, ou un `OR` imbriqué qui renvoie plus de lignes qu'une valeur simple prouve que la forme du filtre est contrôlée par l'attaquant. Le correctif consiste à valider et choisir des champs et opérateurs spécifiques plutôt que d'étaler la requête, donc un endpoint qui transmet l'objet brut est le schéma à rechercher.

## Outils

- **Burp Repeater** : envoyer des objets opérateur là où un scalaire est attendu et comparer les jeux de résultats.
- **Burp Intruder** : énumérer les champs et opérateurs filtrables que l'endpoint transmet.

## Références

- [Prisma : Filtering and sorting](https://www.prisma.io/docs/orm/prisma-client/queries/filtering-and-sorting)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
