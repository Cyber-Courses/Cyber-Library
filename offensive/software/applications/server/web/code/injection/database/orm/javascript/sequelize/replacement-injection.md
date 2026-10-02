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

# Replacement injection

Sequelize `replacements` substitute `:name`/`?` tokens with **escaped values** before the query runs. That is safe for *values*, but the mechanism is string substitution, not true prepared-statement binding, and several patterns defeat it: concatenating alongside replacements, using replacements for identifiers, and second-order reuse of stored data.

## Vulnerable patterns

**Concatenation next to replacements**, the interpolated part is unprotected:

```js
await sequelize.query(
  "SELECT * FROM users WHERE name = :name AND role = '" + role + "'",
  { replacements: { name } }
);
```

**Interpolated identifier/structural position**, `replacements` escape *values*, so `:col` would be rendered as a quoted literal (`ORDER BY 'sort'`, which is inert). Identifiers such as a column or `ORDER BY` slot therefore **cannot** be parameterized and are commonly **interpolated** instead, and that interpolation is the injectable sink:

```js
// identifier can't be a replacement, so it gets concatenated — injectable
await sequelize.query(`SELECT * FROM users ORDER BY ${req.query.sort}`);
```

**Second-order**, a value stored safely earlier is later concatenated into another raw query without replacements.

## Exploitation

For the concatenated `role` slot, inject as a normal quoted-string context:

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

For the **interpolated** `ORDER BY`/identifier slot there is no quote to escape, so inject an expression directly, recall this only works when the slot is interpolated, since a value passed through `replacements` would be escaped to an inert quoted literal:

```
(CASE WHEN (SELECT 1 FROM users WHERE username='admin' AND SUBSTRING(password,1,1)='a') THEN 1 ELSE 2 END)
```

Second-order payloads are stored benign and triggered when the later concatenating query runs; they bypass input-time inspection entirely. The lesson for exploitation is that the presence of `replacements` does **not** guarantee safety, inspect every slot for whether it is a value (escaped) or a structural/concatenated position (injectable).

## Tools

- **sqlmap**: exploiting the concatenated and interpolated-identifier slots alongside replacements.
- **Burp Repeater**: probing each slot for value versus structural context.

## References

- [Sequelize docs: Raw queries, replacements vs bind](https://sequelize.org/docs/v6/core-concepts/raw-queries/#replacements)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
