---
title: "SQLite schema discovery: sqlite_master and sqlite_schema"
description: Listing tables and indexes from sqlite_master—core reconnaissance for SQLi against SQLite backends.
keywords:
  - sqlite_master
  - sqlite_schema
---
# Schema and table discovery (SQLite)

**`sqlite_master`** (alias **`sqlite_schema`**) stores **DDL** for tables, indexes, views, and triggers. **`SELECT sql FROM sqlite_master`** reveals **create** statements when readable.

## Context

`sqlite_master` holds DDL for tables/indexes/views; `SELECT sql FROM sqlite_master` dumps definitions.
## Technique

Filter `type='table'` for user objects; follow with `pragma_table_info` for columns.
## Practice

- Attached DBs: `pragma database_list` then repeat per file.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Enumeration techniques (SQLite)](index.md)
