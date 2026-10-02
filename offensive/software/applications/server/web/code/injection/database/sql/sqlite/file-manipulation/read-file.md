---
title: "SQLite read file context: attached databases and schema disclosure"
description: Reading content indirectly via sqlite_master and ATTACH—path traversal risk when SQL can change ATTACH targets.
keywords:
  - ATTACH
  - sqlite_master
---
# Read file (SQLite)

SQLite “**read file**” in web contexts often means **reading another database file** via **`ATTACH DATABASE`** or **inferring** file-backed state from **`PRAGMA database_list`**. **Path** control is essential.

## Context

`ATTACH DATABASE '/path/file.db' AS x` then query `x.sqlite_master`—path traversal if the app builds ATTACH from user input.
## Technique

Also read sensitive rows once the DB file is world-readable on the host.
## Practice

- Pair with LFI elsewhere to place an attachable file.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [File manipulation (SQLite)](index.md)
