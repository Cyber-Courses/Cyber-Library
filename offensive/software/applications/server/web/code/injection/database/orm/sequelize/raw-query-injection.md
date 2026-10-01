---
title: "Sequelize raw query injection: sequelize.query() with concatenated SQL"
description: sequelize.query() runs raw SQL against the backend; concatenating request data into it instead of using replacements or bind yields SQL injection in Node.js apps.
keywords:
  - Sequelize
  - raw query injection
  - sequelize.query()
  - Node.js ORM
  - replacements
  - SQL injection
---

# Sequelize raw query injection

`sequelize.query()` executes raw SQL against the configured dialect (PostgreSQL, MySQL/MariaDB, SQLite, SQL Server). It accepts `replacements` and `bind` options for safe parameterization, but when application code concatenates request data into the SQL string, none of that applies and the call is a SQL injection sink.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable pattern

```js
// name from the request
const rows = await sequelize.query(
  "SELECT * FROM users WHERE name = '" + name + "'"
);
await sequelize.query(`SELECT * FROM users WHERE id = ${req.query.id}`);
```

The safe forms are `{ replacements: { name } }` with `:name`, or `{ bind: [name] }` with `$1`; the injectable form interpolates into the string.

## Exploitation

Standard injection in the value's context. Numeric `id`:

```
1 OR 1=1
0 UNION SELECT id, username, password FROM users
```

Quoted `name`:

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

Dialect matters for weaponization: the **MySQL** (`mysql2`) driver and **MSSQL** often allow **stacked queries** by default, enabling `; UPDATE`/`; INSERT` after the `SELECT`; PostgreSQL via `pg` can also run multiple statements in a simple query. Use `SLEEP()`/`pg_sleep()` for time-based blind and dialect string functions for substring extraction. When `query()` is called with `{ type: QueryTypes.SELECT }` the rows are returned directly, making `UNION` read straightforward.

## References

- [Sequelize docs: Raw queries](https://sequelize.org/docs/v6/core-concepts/raw-queries/)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
