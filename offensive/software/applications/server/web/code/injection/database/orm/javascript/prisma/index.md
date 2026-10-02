---
title: "Prisma injection"
description: "Prisma Client parameterizes its typed queries and its tagged-template raw API, so injection lives at the Unsafe raw methods and where untrusted objects are spread into a where argument."
keywords:
  - Prisma
  - queryRawUnsafe
  - executeRawUnsafe
  - Prisma where
  - filter injection
---

# Prisma

Prisma Client is parameterized by default. The typed query methods (`findMany`, `update`, and the rest) compile to bound SQL, and the tagged-template raw API (`$queryRaw` and `$executeRaw`) parameterizes its interpolations. Injection appears in two specific places: the explicit `Unsafe` raw methods, which take a plain string, and `where`/filter arguments that are built from an untrusted object.

## Where it goes wrong

- **[Raw query injection](raw-query-injection.md)**: `$queryRawUnsafe` and `$executeRawUnsafe` accept a string, so input concatenated into them injects SQL. The safe tagged-template `$queryRaw` is a short edit away, and mixing the two is the mistake.
- **[Filter injection](filter-injection.md)**: spreading a request object into a `where` lets the caller supply Prisma operators and relation filters, changing which rows match beyond what the endpoint intended.

## The safe and unsafe paths side by side

Prisma makes the boundary unusually visible: the dangerous methods carry `Unsafe` in their names. `$queryRaw\`...\`` binds every `${}`; `$queryRawUnsafe(str)` runs `str` as given. The subtler case is the typed API itself, which is injection-safe for values but trusts the shape of the `where` object, so an endpoint that forwards attacker JSON into it is handing the caller control of the query structure rather than just a value.

## Tools

- **sqlmap**: exploiting $queryRawUnsafe and $executeRawUnsafe sinks.
- **Burp Repeater**: testing raw methods and untrusted objects spread into where.

## References

- [Prisma: Raw queries](https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
