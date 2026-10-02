---
title: "Drizzle injection"
description: "Drizzle's sql template binds its interpolations as parameters and its operator functions are value-safe, so injection appears through sql.raw and raw fragments, and through filters whose column or operator comes from input."
keywords:
  - Drizzle
  - sql.raw
  - Drizzle sql template
  - operator injection
  - raw fragment
---

# Drizzle

Drizzle is parameterized on its normal paths: the `sql` tagged template binds every interpolation, and the operator helpers (`eq`, `gt`, `inArray`, and the rest) pass values as parameters. Injection appears where code drops to a raw fragment, or where the structure of a filter, its column or operator, is chosen from untrusted input.

## Where it goes wrong

- **[Raw SQL injection](raw-sql-injection.md)**: `sql.raw()` and strings concatenated inside a fragment reach the database unparameterized.
- **[Operator injection](operator-injection.md)**: selecting a column or operator from input, or spreading an untrusted filter, changes which rows match beyond the intended predicate.

## The safe and unsafe paths

The `sql` template is the safe primitive: `sql\`... ${value} ...\`` binds `value`. The escape hatch is `sql.raw(string)`, which inserts the string literally, intended for trusted dynamic SQL and dangerous the moment the string carries input. The operator functions are safe for the values they compare, but Drizzle lets a query be composed dynamically, so code that decides which column or which operator to use from the request is handing the caller part of the query shape. The template and the operator values are safe; `sql.raw`, concatenation inside a fragment, and input-chosen columns or operators are the sinks.

## Tools

- **sqlmap**: exploiting sql.raw and concatenated-fragment sinks.
- **Burp Repeater**: testing input-chosen columns, operators, and raw fragments.

## References

- [Drizzle: Magic sql operator](https://orm.drizzle.team/docs/sql)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
