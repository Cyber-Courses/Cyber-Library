---
title: "Drizzle operator injection"
description: "Building a Drizzle where clause by choosing a column or operator from untrusted input, or spreading an untrusted filter, lets a caller change which rows match beyond the intended predicate."
keywords:
  - Drizzle where
  - operator injection
  - dynamic filter
  - column selection
  - query logic
---

# Operator injection

Drizzle's operator helpers compare values safely, but the query is composed in application code, and that composition can itself take untrusted input. When the column being filtered, the operator applied, or the whole filter object comes from the request, the caller controls the query's logic even though every value is still bound as a parameter.

## Column chosen from input

```js
// UNSAFE: the filtered column is attacker-controlled
const col = users[req.query.field];          // field picked by the caller
const rows = await db.select().from(users).where(eq(col, req.query.value));
```

If `field` can name any column, the caller filters on columns the endpoint never meant to expose, and where the lookup falls through to a raw fragment or an unknown key, it can break the query shape entirely. The operator can be attacker-chosen the same way, swapping an intended `eq` for a `like` or a range that returns far more rows.

## Spreading an untrusted filter

Code that assembles conditions from a request object, or forwards it into a dynamic `and`/`or` builder, lets the caller supply the structure:

```js
// UNSAFE: conditions built from arbitrary request keys
const conditions = Object.entries(req.body.filter).map(([k, v]) => eq(users[k], v));
const rows = await db.select().from(users).where(and(...conditions));
```

A body naming privileged columns (`isAdmin`, `role`) filters on them; a body whose keys miss the table map produces `eq(undefined, v)` and either errors informatively or, if combined with a raw fallback, injects.

## Confirming the flaw

Vary the `field`/operator and the filter keys and watch the result set and errors: filtering on a column the endpoint never offered, or flipping the operator to widen the match, shows the query structure is caller-controlled. The safe form selects the column and operator from a fixed allowlist and validates the filter keys, so an endpoint that maps request keys straight onto columns is the pattern to find.

## Tools

- **Burp Repeater**: varying the field, operator, and filter keys and comparing result sets.
- **Burp Intruder**: enumerating column names accepted by the dynamic filter.

## References

- [Drizzle: Filters](https://orm.drizzle.team/docs/operators)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
