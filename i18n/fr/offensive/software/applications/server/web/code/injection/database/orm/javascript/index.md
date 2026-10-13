---
order: 1
title: "Injection ORM JavaScript"
description: "Les ORM Node et TypeScript (Sequelize, Prisma, Drizzle, TypeORM) paramètrent par défaut, mais leurs échappatoires de requête brute et les objets opérateur ou filtre contrôlés par l'attaquant rouvrent l'injection SQL et l'abus de logique de requête."
keywords:
  - ORM JavaScript
  - injection ORM Node.js
  - Prisma
  - Drizzle
  - Sequelize
---

# JavaScript

Les back ends JavaScript et TypeScript se tournent vers un ORM ou un query builder, et chacun embarque un chemin par défaut sûr et paramétré. L'injection apparaît là où l'application quitte ce chemin : une méthode de requête brute qui prend une chaîne simple, ou un objet pris dans l'entrée de requête et étalé dans un argument `where` ou filtre où l'ORM interprète ses clés comme des opérateurs.

## Les ORM ici

- **[Sequelize](sequelize/index.md)** : `sequelize.query()` brut, identifiants interpolés et objets opérateur contrôlés par l'attaquant.
- **[Prisma](prisma/index.md)** : les échappatoires `$queryRawUnsafe` / `$executeRawUnsafe`, et les objets non fiables étalés dans `where`.
- **[Drizzle](drizzle/index.md)** : `sql.raw()` et fragments bruts, et filtres dynamiques construits à partir de l'entrée.

## Le schéma commun

Deux points d'injection reviennent chez tous. Le premier est l'**échappatoire brute** : une méthode qui accepte une chaîne au lieu d'un template paramétré, de sorte que l'entrée concaténée devient du SQL. Le second est l'**injection d'opérateur ou de filtre** : parce que ces ORM expriment les prédicats de requête sous forme d'objets simples, un endpoint qui étale un corps JSON dans un filtre laisse l'appelant fournir des clés d'opérateur (`not`, `gt`, filtres de relation imbriqués) qui élargissent ou contournent la requête prévue sans aucun SQL brut. L'API de requête typée et le template paramétré sont sûrs ; la méthode suffixée `Unsafe`, le fragment brut et l'objet filtre non validé sont où regarder.

## Outils

- **sqlmap** : exploiter les échappatoires de requête brute dans Sequelize, Prisma et Drizzle.
- **Burp Repeater et Intruder** : façonner les charges utiles SQL brut et les objets opérateur ou filtre.

## Références

- [OWASP : SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
