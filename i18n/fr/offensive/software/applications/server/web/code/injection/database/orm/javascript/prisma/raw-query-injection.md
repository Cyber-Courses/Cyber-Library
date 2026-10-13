---
order: 1
title: "Injection de requête brute Prisma"
description: "queryRawUnsafe et executeRawUnsafe prennent une chaîne simple, donc l'entrée non fiable concaténée dedans s'exécute comme du SQL ; le queryRaw à template taggé paramètre et est la forme sûre."
keywords:
  - queryRawUnsafe
  - executeRawUnsafe
  - SQL brut Prisma
  - template taggé
  - injection SQL
---

# Injection de requête brute

Prisma expose deux familles de méthodes de requête brute. Les formes à template taggé, `$queryRaw` et `$executeRaw`, paramètrent chaque interpolation. Les formes à chaîne, `$queryRawUnsafe` et `$executeRawUnsafe`, prennent une chaîne simple et l'exécutent telle quelle. Lorsque l'entrée est concaténée dans la forme `Unsafe`, c'est une injection SQL sans aucune des protections de Prisma.

## Le point d'injection

```js
// UNSAFE: input concatenated into a string method
const rows = await prisma.$queryRawUnsafe(
  `SELECT * FROM "User" WHERE email = '${req.query.email}'`
);
```

Fournir `' OR '1'='1` pour `email` renvoie toutes les lignes, et `'; DROP TABLE "User"; --` exécute une instruction empilée là où le pilote le permet. `$executeRawUnsafe` est le même danger pour les écritures.

## La forme sûre qui aurait dû être utilisée

Le template taggé lie la valeur comme paramètre, donc la requête d'apparence identique est sûre :

```js
// SAFE: tagged template parameterizes ${}
const rows = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE email = ${req.query.email}
`;
```

Les deux sont à un mot d'écart, ce qui explique pourquoi l'appel `Unsafe` est souvent choisi lorsqu'une requête doit être construite dynamiquement puis alimentée par une entrée concaténée.

## Les fragments bruts déjouent aussi la méthode sûre

Les méthodes `Unsafe` ne sont pas la seule voie d'entrée. Le `$queryRaw` à template taggé accepte des fragments `Sql` construits avec `Prisma.raw()` (et `Prisma.sql`), et `Prisma.raw()` insère sa chaîne telle quelle, sans paramétrage. Insérer une chaîne de l'attaquant via `Prisma.raw()` injecte même si l'appel externe est le `$queryRaw` sûr :

```js
// UNSAFE: Prisma.raw inside the safe tagged template is not parameterized
const rows = await prisma.$queryRaw(
  Prisma.sql`SELECT * FROM "User" WHERE email = ${Prisma.raw("'" + req.query.email + "'")}`
);
```

Donc le point d'injection est tout texte brut non paramétré, qu'il arrive via `$queryRawUnsafe` ou via un fragment `Prisma.raw()` passé à `$queryRaw` ; seule une valeur interpolée directement comme `${}` est liée.

## SQL dynamique et identifiants

`$queryRawUnsafe` accepte aussi des paramètres positionnels (`$queryRawUnsafe(query, ...values)`), et le code qui les utilise pour les valeurs mais concatène tout de même un identifiant, un nom de table, une colonne, une direction `ORDER BY`, reste injectable via la partie concaténée, parce que les identifiants ne peuvent pas être paramétrés. Tout endroit où la chaîne de requête elle-même est assemblée à partir de l'entrée, plutôt que passée comme valeur liée, est la chose à trouver.

## Outils

- **sqlmap** : automatiser l'extraction contre les points d'injection $queryRawUnsafe et Prisma.raw.
- **Burp Repeater et Intruder** : délivrer des charges utiles d'évasion et d'instructions empilées.

## Références

- [Prisma : Raw queries](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries)
- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
