---
title: "SQL injection in web applications: unsafe query construction, ORM edge cases, and second-order SQLi"
description: Application-layer SQL string construction and bind mistakes that let attacker-controlled data alter query semantics.
keywords:
  - SQL injection
  - prepared statement
  - second order SQLi
  - ORM
---

# SQL injection

**SQL injection** is the unsafe mixing of user or external data into a query string the database interprets as syntax or side effects, not only as a literal value. It remains critical because a single mistake in reporting code, an `ORDER BY` builder, or a raw `execute()` path can return arbitrary rows, modify data, or, on some stacks, reach extended features exposed through SQL.

**Engine-specific patterns** (dialect functions, file reads, out-of-band channels) live under per-engine folders below.

## By engine (library structure)

| Engine | Notes |
|--------|--------|
| [IBM Db2](ibm-db2/index.md) | Blind, enumeration, error-based, command execution, DIOS-style aggregation, WAF-oriented leaves |
| [Microsoft SQL Server](mssql/index.md) | Union, blind, time-based, error-based, stacked queries, file, OOB, privileges, linked servers |
| [MySQL](mysql/index.md) | Union-based, blind, error-based, time-based, and related leaves |
| [Oracle](oracle/index.md) | Union, blind, PL/SQL, file and scheduler surfaces, OOB (UTL packages), XML/XXE overlap |
| [PostgreSQL](postgresql/index.md) | Blind (boolean-based leaves), error-based, time-based, file, command execution, WAF, mirrors Library Structure |
| [SQLite](sqlite/index.md) | sqlite_master/PRAGMA enumeration, file attach/export, `load_extension` risk |
