---
order: 1
title: "MongoDB $exists comme oracle booléen pour l'inférence de champs"
description: "L'opérateur $exists teste si un champ est présent, offrant un oracle vrai/faux pour cartographier le schéma des documents et inférer des données lorsqu'aucune ligne n'est reflétée."
keywords:
  - $exists
  - injection MongoDB
  - oracle booléen
  - énumération de champs
  - inférence de schéma
  - injection aveugle
---

# Exists

`$exists` teste si un champ est présent dans un document, renvoyant un booléen pur. Injecté dans un filtre, il devient un oracle pour la forme des données : quels champs existent, sur quels documents, sans qu'aucune valeur ne soit jamais reflétée.

La précondition est que l'objet contrôlé par l'attaquant atteigne le filtre non typé, de sorte que `{"$exists": true}` est analysé comme opérateur plutôt que matché comme donnée.

## Oracle de présence de champ

Associez `$exists` à un sélecteur connu et observez si la requête matche toujours. La différence entre `true` et `false` vous indique si le champ nommé est présent sur le document ciblé :

```json
{"username": "admin", "password": {"$exists": true}}
{"username": "admin", "mfaSecret": {"$exists": true}}
{"username": "admin", "resetToken": {"$exists": true}}
```

Si le premier matche et pas le second, `admin` possède un champ `password` mais pas de `mfaSecret`. Cela cartographie le schéma du document un champ à la fois : sondez des noms candidats (`password`, `passwordHash`, `apiKey`, `ssn`, `isAdmin`, `resetToken`, `stripeCustomerId`) et conservez ceux qui renvoient vrai. Via un analyseur imbriqué :

```
username=admin&resetToken[$exists]=true
```

## Inférence et branchement

Au-delà de la cartographie du schéma, `$exists` est un contrôle propre pour construire des chaînes d'inférence. Comme il renvoie un vrai/faux fiable, il ancre l'étape « y a-t-il quelque chose à extraire ici » avant de basculer vers `$regex` ou les opérateurs de comparaison pour la valeur elle-même :

```json
{"email": "victim@example.com", "resetToken": {"$exists": true}}
```

Un résultat vrai indique qu'un token de réinitialisation actif existe pour ce compte, un signal digne d'action avant de l'extraire caractère par caractère avec [Regex](regex.md). `$exists` construit aussi des branches forcées-vraies et forcées-fausses pour calibrer tout oracle booléen : `{"_id": {"$exists": true}}` matche chaque document (une ligne de base garantie-vraie), tandis que `{"__nope__": {"$exists": true}}` contre un champ qui ne peut exister donne une ligne de base garantie-fausse. Comparer une sonde à ces deux lignes de base distingue un vrai négatif d'une erreur ou d'un résultat vide.

Combiné aux opérateurs de [Opérateurs de comparaison](comparison-operators.md) et [Opérateurs de tableau](array-operators.md), `$exists` décide d'abord où vivent les données, puis ces opérateurs les lisent.

## Outils

- **Burp Repeater** : soumettre des sondes $exists pour cartographier quels champs sont présents.
- **nosqli** : scanner les paramètres à la recherche d'injection d'objet opérateur.
- **NoSQLMap** : automatiser l'énumération de présence de champ.

## Références

- [MongoDB : $exists](https://www.mongodb.com/docs/manual/reference/operator/query/exists/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
