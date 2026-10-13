---
order: 2
title: "Injection d'opérateur Drizzle"
description: "Construire une clause where Drizzle en choisissant une colonne ou un opérateur à partir d'une entrée non fiable, ou en étalant un filtre non fiable, laisse un appelant changer les lignes qui correspondent au-delà du prédicat prévu."
keywords:
  - Drizzle where
  - injection d'opérateur
  - filtre dynamique
  - sélection de colonne
  - logique de requête
---

# Injection d'opérateur

Les helpers d'opérateur de Drizzle comparent les valeurs en toute sécurité, mais la requête est composée dans le code applicatif, et cette composition peut elle-même prendre une entrée non fiable. Lorsque la colonne filtrée, l'opérateur appliqué ou l'objet filtre entier provient de la requête, l'appelant contrôle la logique de la requête même si chaque valeur reste liée comme paramètre.

## Colonne choisie depuis l'entrée

```js
// UNSAFE: the filtered column is attacker-controlled
const col = users[req.query.field];          // field picked by the caller
const rows = await db.select().from(users).where(eq(col, req.query.value));
```

Si `field` peut nommer n'importe quelle colonne, l'appelant filtre sur des colonnes que l'endpoint n'a jamais prévu d'exposer, et là où la recherche retombe sur un fragment brut ou une clé inconnue, cela peut casser entièrement la forme de la requête. L'opérateur peut être choisi par l'attaquant de la même manière, remplaçant un `eq` prévu par un `like` ou une plage qui renvoie beaucoup plus de lignes.

## Étalement d'un filtre non fiable

Le code qui assemble des conditions à partir d'un objet de requête, ou le transmet dans un constructeur `and`/`or` dynamique, laisse l'appelant fournir la structure :

```js
// UNSAFE: conditions built from arbitrary request keys
const conditions = Object.entries(req.body.filter).map(([k, v]) => eq(users[k], v));
const rows = await db.select().from(users).where(and(...conditions));
```

Un corps nommant des colonnes privilégiées (`isAdmin`, `role`) filtre sur elles ; un corps dont les clés ne figurent pas dans le mappage de table produit `eq(undefined, v)` et soit lève une erreur informative, soit, s'il est combiné à un repli brut, injecte.

## Confirmer la faille

Faites varier le `field`/l'opérateur et les clés de filtre et observez le jeu de résultats et les erreurs : filtrer sur une colonne que l'endpoint n'a jamais offerte, ou retourner l'opérateur pour élargir la correspondance, montre que la structure de la requête est contrôlée par l'appelant. La forme sûre sélectionne la colonne et l'opérateur dans une liste blanche fixe et valide les clés de filtre, donc un endpoint qui mappe les clés de requête directement sur les colonnes est le schéma à trouver.

## Outils

- **Burp Repeater** : faire varier le champ, l'opérateur et les clés de filtre et comparer les jeux de résultats.
- **Burp Intruder** : énumérer les noms de colonne acceptés par le filtre dynamique.

## Références

- [Drizzle : Filters](https://orm.drizzle.team/docs/operators)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
