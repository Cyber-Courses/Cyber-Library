---
title: "SQLite SQL injection: techniques and payloads"
description: "Exploiting SQL injection against SQLite: sqlite_master enumeration, loose typing, real error primitives, randomblob timing, ATTACH file writes, and load_extension RCE."
keywords:
  - SQLite SQL injection
  - sqlite_master
  - load_extension
  - ATTACH DATABASE
  - randomblob
  - embedded database
---

# SQLite

SQLite is an embedded, file-based database with no server or user accounts, so injection against it has a different shape from the client-server engines. Several traits matter.

There is no `information_schema`; the schema lives in `sqlite_master` (aliased `sqlite_schema` in newer versions), whose `sql` column stores each object's `CREATE` statement and therefore reveals column names directly. There is no `SLEEP`, so time-based inference uses a deliberately expensive expression such as `randomblob()`. SQLite is loosely typed, which makes `UNION` easy (type mismatches rarely block it) but breaks some error tricks from other engines: dividing by zero returns `NULL` rather than erroring, and casting a non-numeric string yields `0`, so error-based injection must use primitives that genuinely raise an error.

Comments are `--` and `/* */`. String concatenation is `||`. Stacked queries depend on the API: `sqlite3_exec` runs multiple `;`-separated statements, while a prepared statement runs only the first, so stacking is confirmed, not assumed.

The high-impact primitives are `ATTACH DATABASE`, which can write a file, and `load_extension()`, which loads a shared library for code execution but is disabled by default. There are no privilege levels inside SQLite: whatever the application's database handle can do, the injection can do.

## Techniques

- **[Enumeration](enumeration.md)**: version and schema from `sqlite_master`.
- **[Authentication bypass](authentication-bypass.md)**: subvert a login built from the credential fields.
- **[Union-based](union-based.md)**: append a `UNION SELECT`, aided by loose typing.
- **[Error-based](error-based.md)**: leak values through primitives that actually raise errors.
- **[Blind](blind.md)**: infer data from boolean response differences.
- **[Time-based](time-based.md)**: infer data with `randomblob()` heavy expressions.
- **[File manipulation](file-manipulation.md)**: write files with `ATTACH` and `VACUUM INTO`.
- **[Remote code execution](remote-code-execution.md)**: `load_extension()` and the ATTACH web-shell.
- **[Evasion techniques](evasion-techniques.md)**: comments, hex, and encoding past filters.

## References

- SQLite Documentation: `sqlite_master`, core functions, ATTACH, load_extension
- OWASP Testing Guide: Testing for SQL Injection
