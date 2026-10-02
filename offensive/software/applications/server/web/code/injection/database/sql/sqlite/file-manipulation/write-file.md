---
title: "SQLite write file: export, backup, and crafted database files"
description: Writing artifacts through SQLite backup/export features, overlap with application file upload and web root placement.
keywords:
  - backup
  - export
---
# Write file (SQLite)

**Write** paths include **`VACUUM INTO`**, **backup APIs**, or application-level **export** that dumps **SQL** or **binary** DBs to attacker-influenced paths.

## Context

`VACUUM INTO`, backup APIs, or export features write `.db` or SQL dumps, weaponize if you control destination path.
## Technique

Webshell via `SELECT '<?php ...' INTO OUTFILE` style only exists when the engine exposes file output primitives, SQLite itself is usually attach/dump oriented.
## Practice

- Map app export features that shell out to `sqlite3 .dump`.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
