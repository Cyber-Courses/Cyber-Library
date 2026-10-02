---
title: "Entity Framework Core injection"
description: "Injection through FromSqlRaw/ExecuteSqlRaw and the interpolated-string footgun in EF Core."
keywords:
  - Entity Framework
  - EF Core
  - FromSqlRaw
  - string interpolation
  - .NET
---

# Entity Framework

Entity Framework Core parameterizes LINQ queries, but its raw-SQL family, `FromSqlRaw`, `ExecuteSqlRaw`, `SqlQueryRaw`, runs whatever string it is given, and handing a C# interpolated string to a `*Raw` method bakes user input into the SQL before EF ever sees it.

## Tools

- **sqlmap**: exploiting FromSqlRaw and ExecuteSqlRaw sinks against the backend.
- **Burp Repeater**: hand-crafting payloads for the raw and interpolated-string sinks.

## References

- Microsoft EF Core documentation: Raw SQL queries
- OWASP: SQL Injection
