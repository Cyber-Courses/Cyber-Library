---
title: "SQL injection by database engine"
description: "SQL injection organized by database engine, because the exploitable syntax, functions, catalog, and file and command primitives differ sharply between MySQL, PostgreSQL, SQL Server, Oracle, SQLite, and Db2."
keywords:
  - SQL injection
  - database engine dialects
  - SQLi techniques
  - information_schema
  - database exploitation
---

# SQL

SQL injection happens when untrusted input is concatenated into a SQL query instead of being bound as a parameter, letting an attacker change the query's meaning. The high-level techniques are shared, union-based reads, error-based leaks, boolean and time-based blind inference, out-of-band exfiltration, and file or command access, but the exact payload that implements each one is engine-specific.

That is why this area is organized by database engine rather than by technique alone. Comment syntax, string concatenation, the system catalog, timing functions, and the primitives for reading files or running commands all differ between engines. A payload that works verbatim on MySQL fails on PostgreSQL or Oracle, so knowing which engine is behind the application decides which syntax applies. Each engine page covers fingerprinting it and then works through the techniques in its own dialect.

## Engines

- **[MySQL](mysql/index.md)** (and the compatible MariaDB): `information_schema`, XPath error functions, `SLEEP`, and `FILE`-gated read and write.
- **PostgreSQL**: dollar-quoting, `CAST` error leaks, `pg_sleep`, and `COPY ... PROGRAM` for command execution.
- **Microsoft SQL Server**: stacked queries, conversion-error leaks, `WAITFOR DELAY`, and `xp_cmdshell`.
- **Oracle**: mandatory `FROM DUAL`, `UTL_*` packages, and ACL-gated out-of-band channels.
- **SQLite**: `sqlite_master` enumeration, no `information_schema`, and API-dependent stacked queries.
- **IBM Db2**: `SYSIBM.SYSDUMMY1`, special registers, and `SYSCAT` catalog views.

## References

- OWASP Testing Guide: Testing for SQL Injection
- PortSwigger Web Security Academy: SQL injection
