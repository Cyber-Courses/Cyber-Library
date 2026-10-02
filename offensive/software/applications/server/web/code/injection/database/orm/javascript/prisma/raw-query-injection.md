---
title: "Prisma raw query injection"
description: "queryRawUnsafe and executeRawUnsafe take a plain string, so untrusted input concatenated into them runs as SQL; the tagged-template queryRaw parameterizes and is the safe form."
keywords:
  - queryRawUnsafe
  - executeRawUnsafe
  - Prisma raw SQL
  - tagged template
  - SQL injection
---

# Raw query injection

Prisma exposes two families of raw query methods. The tagged-template forms, `$queryRaw` and `$executeRaw`, parameterize every interpolation. The string forms, `$queryRawUnsafe` and `$executeRawUnsafe`, take a plain string and run it verbatim. When input is concatenated into the `Unsafe` form, it is SQL injection with none of Prisma's protection.

## The sink

```js
// UNSAFE: input concatenated into a string method
const rows = await prisma.$queryRawUnsafe(
  `SELECT * FROM "User" WHERE email = '${req.query.email}'`
);
```

Supplying `' OR '1'='1` for `email` returns every row, and `'; DROP TABLE "User"; --` runs a stacked statement where the driver permits it. `$executeRawUnsafe` is the same hazard for writes.

## The safe form it should have been

The tagged template binds the value as a parameter, so the identical-looking query is safe:

```js
// SAFE: tagged template parameterizes ${}
const rows = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE email = ${req.query.email}
`;
```

The two are one word apart, which is why the `Unsafe` call is often reached for when a query needs to be built dynamically and then fed concatenated input.

## Raw fragments defeat the safe method too

The `Unsafe` methods are not the only way in. The tagged-template `$queryRaw` accepts `Sql` fragments built with `Prisma.raw()` (and `Prisma.sql`), and `Prisma.raw()` inserts its string verbatim, unparameterized. Splicing an attacker string through `Prisma.raw()` injects even though the outer call is the safe `$queryRaw`:

```js
// UNSAFE: Prisma.raw inside the safe tagged template is not parameterized
const rows = await prisma.$queryRaw(
  Prisma.sql`SELECT * FROM "User" WHERE email = ${Prisma.raw("'" + req.query.email + "'")}`
);
```

So the sink is any unparameterized raw text, whether it arrives through `$queryRawUnsafe` or through a `Prisma.raw()` fragment handed to `$queryRaw`; only a value interpolated directly as `${}` is bound.

## Dynamic SQL and identifiers

`$queryRawUnsafe` also accepts positional parameters (`$queryRawUnsafe(query, ...values)`), and code that uses those for values but still concatenates an identifier, a table name, a column, an `ORDER BY` direction, remains injectable through the concatenated part, because identifiers cannot be parameterized. Any place the query string itself is assembled from input, rather than passed as a bound value, is the thing to find.

## References

- [Prisma: Raw queries](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
