---
title: "JavaScript ORM injection"
description: "Node and TypeScript ORMs (Sequelize, Prisma, Drizzle, TypeORM) parameterize by default, but their raw-query escape hatches and attacker-controlled operator or filter objects reopen SQL injection and query-logic abuse."
keywords:
  - JavaScript ORM
  - Node.js ORM injection
  - Prisma
  - Drizzle
  - Sequelize
---

# JavaScript

JavaScript and TypeScript back ends reach for an ORM or query builder, and each ships a safe, parameterized default path. Injection appears where the application steps off that path: a raw-query method that takes a plain string, or an object taken from request input and spread into a `where` or filter argument where the ORM interprets its keys as operators.

## The ORMs here

- **[Sequelize](sequelize/index.md)**: raw `sequelize.query()`, interpolated identifiers, and attacker-controlled operator objects.
- **[Prisma](prisma/index.md)**: the `$queryRawUnsafe` / `$executeRawUnsafe` escape hatches, and untrusted objects spread into `where`.
- **[Drizzle](drizzle/index.md)**: `sql.raw()` and raw fragments, and dynamic filters built from input.

## The shared pattern

Two sinks recur across all of them. The first is the **raw escape hatch**: a method that accepts a string instead of a parameterized template, so concatenated input becomes SQL. The second is **operator or filter injection**: because these ORMs express query predicates as plain objects, an endpoint that spreads a JSON body into a filter lets the caller supply operator keys (`not`, `gt`, nested relation filters) that widen or bypass the intended query without any raw SQL at all. The typed query API and the parameterized template are safe; the `Unsafe`-suffixed method, the raw fragment, and the unvalidated filter object are where to look.

## Tools

- **sqlmap**: exploiting the raw-query escape hatches in Sequelize, Prisma, and Drizzle.
- **Burp Repeater and Intruder**: crafting raw-SQL payloads and operator or filter objects.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
