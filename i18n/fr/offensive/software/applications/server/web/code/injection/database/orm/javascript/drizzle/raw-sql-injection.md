---
order: 1
title: "Injection SQL brute Drizzle"
description: "Le template sql de Drizzle paramètre ses interpolations, mais sql.raw() et la construction de chaîne à l'intérieur d'un fragment atteignent la base de données sans paramétrage lorsqu'ils sont alimentés par une entrée non fiable."
keywords:
  - Drizzle sql.raw
  - fragment brut
  - template sql
  - injection SQL
  - espace réservé
---

# Injection SQL brute

Le template `sql` de Drizzle est sûr : chaque interpolation `${}` est envoyée comme paramètre lié. La méthode `sql.raw()` est l'inverse, elle insère son argument dans la requête tel quel, sans binding, pour les cas où le texte SQL lui-même doit être construit dynamiquement. Une entrée non fiable atteignant `sql.raw()`, ou concaténée dans un fragment, est une injection SQL directe.

## Le point d'injection

```js
// UNSAFE: input placed into the query text with sql.raw
const rows = await db.execute(
  sql`SELECT * FROM users WHERE email = ${sql.raw("'" + req.query.email + "'")}`
);

// UNSAFE: concatenation building the fragment text
const rows2 = await db.execute(sql.raw(`SELECT * FROM users WHERE id = ${req.query.id}`));
```

`sql.raw` contourne le paramétrage que le template environnant aurait fourni, donc `' OR '1'='1` et les instructions empilées s'appliquent exactement comme dans du SQL écrit à la main.

## La forme sûre

Interpoler la valeur directement dans le template la lie :

```js
// SAFE: the template parameterizes ${}
const rows = await db.execute(sql`SELECT * FROM users WHERE email = ${req.query.email}`);
```

## Quand le SQL dynamique est réellement nécessaire

`sql.raw` existe pour les parties qui ne peuvent pas être paramétrées, un identifiant, une direction de tri, une clause entière choisie à l'exécution. Ces parties doivent provenir d'une liste blanche fixe, non de l'entrée, parce qu'elles sont le seul endroit où le binding ne peut pas protéger. Un dérapage courant consiste à paramétrer correctement la valeur mais à construire le `ORDER BY column direction` à partir de la requête avec `sql.raw`, ce qui laisse une injection dans la clause d'ordonnancement même si le filtre est sûr. Tout `sql.raw` dont l'argument remonte à l'entrée est la chose à trouver.

## Outils

- **sqlmap** : automatiser l'extraction une fois que l'entrée atteint sql.raw.
- **Burp Repeater et Intruder** : délivrer des charges utiles d'évasion et d'instructions empilées.

## Références

- [Drizzle : Magic sql operator](https://orm.drizzle.team/docs/sql)
- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
