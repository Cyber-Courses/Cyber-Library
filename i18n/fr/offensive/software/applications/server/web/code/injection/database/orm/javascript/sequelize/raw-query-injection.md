---
title: "Injection de requête brute Sequelize : sequelize.query() avec SQL concaténé"
description: sequelize.query() exécute du SQL brut contre le backend ; concaténer des données de requête dedans au lieu d'utiliser replacements ou bind donne une injection SQL dans les applications Node.js.
keywords:
  - Sequelize
  - injection de requête brute
  - sequelize.query()
  - ORM Node.js
  - replacements
  - injection SQL
---

# Injection de requête brute

`sequelize.query()` exécute du SQL brut contre le dialecte configuré (PostgreSQL, MySQL/MariaDB, SQLite, SQL Server). Il accepte les options `replacements` et `bind` pour un paramétrage sûr, mais lorsque le code applicatif concatène des données de requête dans la chaîne SQL, rien de cela ne s'applique et l'appel est un point d'injection SQL.

## Schéma vulnérable

```js
// name from the request
const rows = await sequelize.query(
  "SELECT * FROM users WHERE name = '" + name + "'"
);
await sequelize.query(`SELECT * FROM users WHERE id = ${req.query.id}`);
```

Les formes sûres sont `{ replacements: { name } }` avec `:name`, ou `{ bind: [name] }` avec `$1` ; la forme injectable interpole dans la chaîne.

## Exploitation

Injection standard dans le contexte de la valeur. `id` numérique :

```
1 OR 1=1
0 UNION SELECT id, username, password FROM users
```

`name` entre apostrophes :

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

Le dialecte compte pour l'armement de l'attaque. **MSSQL** (`tedious`) exécute des **requêtes empilées** (`; UPDATE`/`; INSERT` après le `SELECT`), et PostgreSQL via `pg` peut exécuter plusieurs instructions dans une seule requête simple. Le pilote **MySQL** `mysql2` **désactive** les instructions multiples par défaut, donc les charges utiles empilées avec `;` ne se déclenchent que lorsque l'application définit explicitement `multipleStatements: true` ; contre une application par défaut adossée à MySQL, repliez-vous sur `UNION` et l'inférence booléenne/temporisée. Utilisez `SLEEP()`/`pg_sleep()` pour l'aveugle temporisé et les fonctions de chaîne du dialecte pour l'extraction par sous-chaîne. Lorsque `query()` est appelé avec `{ type: QueryTypes.SELECT }`, les lignes sont renvoyées directement, rendant la lecture par `UNION` simple.

## Outils

- **sqlmap** : automatiser l'extraction contre les appels sequelize.query() concaténés.
- **Burp Repeater et Intruder** : délivrer des charges utiles UNION et en aveugle spécifiques au dialecte.

## Références

- [Documentation Sequelize : Raw queries](https://sequelize.org/docs/v6/core-concepts/raw-queries/)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
