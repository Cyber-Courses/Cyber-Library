---
title: "Stacked queries in MSSQL injection"
description: "Running additional statements after a semicolon in SQL Server injection, which drivers commonly allow, enabling EXEC, DDL, and configuration changes."
keywords:
  - stacked queries
  - semicolon injection
  - EXEC
  - sp_executesql
  - MSSQL multiple statements
---

# Stacked queries

SQL Server readily executes stacked queries: a `;`-separated second statement usually runs, and many T-SQL drivers (including common ADO.NET and PHP sqlsrv usage) allow it. This turns a read-only-looking injection into arbitrary statement execution, which is what makes the strongest MSSQL primitives reachable.

Append a statement that acts:

```sql
'; UPDATE users SET is_admin=1 WHERE username='attacker'-- 
'; EXEC xp_cmdshell 'whoami'-- 
```

Stacking is what enables `EXEC` of stored and extended procedures, the `sp_configure`/`RECONFIGURE` pair that re-enables `xp_cmdshell`, `WAITFOR` timing, and DDL. Dynamic SQL via `EXEC()` or `sp_executesql` can also be chained, which helps assemble commands from extracted values (building a UNC path for out-of-band, for example).

Availability still depends on the driver and API: a parameterized call that sends one statement rejects the second. Confirm stacking by observing a side effect (a changed row, a created object) rather than assuming it. When it is blocked, fall back to in-query techniques (union, error, blind, error-based) that complete within the single original statement.

## Tools

- **sqlmap**: exploits stacked queries with `--technique=S`.
- **sqlcmd**: official client to confirm multi-statement batch execution.

## References

- Microsoft SQL Server Documentation: batches, EXECUTE, sp_executesql
- OWASP Testing Guide: Testing for SQL Injection
