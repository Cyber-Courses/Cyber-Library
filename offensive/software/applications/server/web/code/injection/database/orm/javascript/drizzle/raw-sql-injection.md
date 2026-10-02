---
title: "Drizzle raw SQL injection"
description: "Drizzle's sql template parameterizes its interpolations, but sql.raw() and string building inside a fragment reach the database unparameterized when fed untrusted input."
keywords:
  - Drizzle sql.raw
  - raw fragment
  - sql template
  - SQL injection
  - placeholder
---

# Raw SQL injection

Drizzle's `sql` template is safe: each `${}` interpolation is sent as a bound parameter. The `sql.raw()` method is the opposite, it inserts its argument into the query verbatim, with no binding, for cases where the SQL text itself must be built dynamically. Untrusted input reaching `sql.raw()`, or concatenated into a fragment, is direct SQL injection.

## The sink

```js
// UNSAFE: input placed into the query text with sql.raw
const rows = await db.execute(
  sql`SELECT * FROM users WHERE email = ${sql.raw("'" + req.query.email + "'")}`
);

// UNSAFE: concatenation building the fragment text
const rows2 = await db.execute(sql.raw(`SELECT * FROM users WHERE id = ${req.query.id}`));
```

`sql.raw` bypasses the parameterization the surrounding template would have provided, so `' OR '1'='1` and stacked statements apply exactly as in hand-written SQL.

## The safe form

Interpolating the value directly into the template binds it:

```js
// SAFE: the template parameterizes ${}
const rows = await db.execute(sql`SELECT * FROM users WHERE email = ${req.query.email}`);
```

## When dynamic SQL is genuinely needed

`sql.raw` exists for parts that cannot be parameterized, an identifier, a sort direction, a whole clause chosen at runtime. Those parts must come from a fixed allowlist, not from input, because they are the one place binding cannot protect. A common slip is parameterizing the value correctly but building the `ORDER BY column direction` from the request with `sql.raw`, which leaves an injection in the ordering clause even though the filter is safe. Any `sql.raw` whose argument traces back to input is the thing to find.

## Tools

- **sqlmap**: automating extraction once input reaches sql.raw.
- **Burp Repeater and Intruder**: delivering breakout and stacked-statement payloads.

## References

- [Drizzle: Magic sql operator](https://orm.drizzle.team/docs/sql)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
