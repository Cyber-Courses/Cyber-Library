---
title: "MSSQL union-based SQL injection: column alignment, NULL padding, and TOP"
description: Union-based extraction in SQL Server—matching column types and counts, using NULL, and pagination with TOP or OFFSET-FETCH.
keywords:
  - UNION ALL
  - MSSQL union SQLi
  - TOP
---
# Union-based (MSSQL)

**UNION** injection requires the attacker to match **column count** and **compatible types**. SQL Server does not use `LIMIT`; **`TOP n`** or **`OFFSET-FETCH`** (2012+) constrains row count. **`NULL`** helps pad unknown columns during probing.

## Context

Union requires matching column count and compatible types; SQL Server uses `TOP`/`OFFSET-FETCH` instead of `LIMIT`.
## Technique

`UNION ALL` avoids duplicate suppression; pad unknown positions with `NULL` or typed literals. Probe column count with `ORDER BY n` until errors flip.
## Practice

- `ORDER BY` column index walks to find arity; then `UNION SELECT NULL,...`.
- Use `@@version`, `db_name()`, `user_name()` for quick wins once columns align.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Error-based](error-based.md)
