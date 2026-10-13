---
order: 2
title: "Injection Prisma"
description: "Prisma Client paramètre ses requêtes typées et son API brute à template taggé, donc l'injection réside dans les méthodes brutes Unsafe et là où des objets non fiables sont étalés dans un argument where."
keywords:
  - Prisma
  - queryRawUnsafe
  - executeRawUnsafe
  - Prisma where
  - injection de filtre
---

# Prisma

Prisma Client est paramétré par défaut. Les méthodes de requête typées (`findMany`, `update`, et les autres) se compilent en SQL lié, et l'API brute à template taggé (`$queryRaw` et `$executeRaw`) paramètre ses interpolations. L'injection apparaît à deux endroits précis : les méthodes brutes explicites `Unsafe`, qui prennent une chaîne simple, et les arguments `where`/filtre construits à partir d'un objet non fiable.

## Où ça dérape

- **[Injection de requête brute](raw-query-injection.md)** : `$queryRawUnsafe` et `$executeRawUnsafe` acceptent une chaîne, donc l'entrée concaténée dedans injecte du SQL. Le `$queryRaw` sûr à template taggé ne demande qu'une courte modification, et mélanger les deux est l'erreur.
- **[Injection de filtre](filter-injection.md)** : étaler un objet de requête dans un `where` laisse l'appelant fournir des opérateurs Prisma et des filtres de relation, changeant les lignes qui correspondent au-delà de ce que l'endpoint prévoyait.

## Les chemins sûrs et non sûrs côte à côte

Prisma rend la frontière exceptionnellement visible : les méthodes dangereuses portent `Unsafe` dans leur nom. `$queryRaw\`...\`` lie chaque `${}` ; `$queryRawUnsafe(str)` exécute `str` tel quel. Le cas plus subtil est l'API typée elle-même, qui est sûre côté injection pour les valeurs mais fait confiance à la forme de l'objet `where`, donc un endpoint qui y transmet du JSON de l'attaquant remet à l'appelant le contrôle de la structure de la requête plutôt que seulement une valeur.

## Outils

- **sqlmap** : exploiter les points d'injection $queryRawUnsafe et $executeRawUnsafe.
- **Burp Repeater** : tester les méthodes brutes et les objets non fiables étalés dans where.

## Références

- [Prisma : Raw queries](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries)
- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
