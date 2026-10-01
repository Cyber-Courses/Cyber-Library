---
title: "C# ORM injection"
description: "Entity Framework and EF Core parameterize LINQ queries, but raw-SQL APIs and unsafe string interpolation into FromSqlRaw and similar sinks reach the database without parameterization."
keywords:
  - C# ORM
  - Entity Framework injection
  - EF Core
  - FromSqlRaw
  - string interpolation
---

# C#

.NET data access runs through Entity Framework and EF Core. LINQ queries translate to parameterized SQL and are safe; the raw-SQL APIs are where injection enters, and C# string interpolation makes the unsafe form look deceptively similar to the safe one.

## The ORM here

- **[Entity Framework](entity-framework/index.md)**: `FromSqlRaw` and `ExecuteSqlRaw` fed concatenated or interpolated strings, versus the parameterizing `FromSqlInterpolated` form.

## The shared pattern

EF Core offers two raw methods that read almost identically: `FromSqlInterpolated` captures an interpolated string as a parameterized `FormattableString` and is safe, while `FromSqlRaw` takes a plain `string` and runs it verbatim. A developer who builds a `$"..."` interpolated string and passes it to `FromSqlRaw` (or concatenates into it) has written injectable code that looks like the safe call. Identifiers interpolated into either method are never parameterized and are always a sink. The tell is `Raw` fed anything the caller influenced.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Microsoft: Raw SQL queries in EF Core](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries)
