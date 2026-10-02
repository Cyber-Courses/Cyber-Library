---
title: "SQLite column discovery: PRAGMA table_info and pragma_table_info"
description: Reading column names and types through PRAGMA interfaces, minimal privilege for app schemas in custom SQLite builds if applicable.
keywords:
  - PRAGMA table_info
  - pragma_table_info
---
# Column and data type discovery (SQLite)

**`PRAGMA table_info(...)`** and **`pragma_table_info`** expose **columns**, **types**, and **constraints** for a table, valuable for **mapping** an injection target.

## Context

`PRAGMA table_info(t)` returns names, types, PK, use for union column planning.
## Technique

`pragma_table_info` table-valued function works in newer SQLite for the same data inside SELECTs.
## Practice

- Join `sqlite_master` to map `sql` DDL strings to inferred types.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
