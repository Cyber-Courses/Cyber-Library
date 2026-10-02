---
title: "SQLite ATTACH DATABASE as a pivot: path control and trust boundaries"
description: Attaching additional database files—risk when SQL injection can supply filesystem paths or influence backup flows.
keywords:
  - ATTACH DATABASE
---
# Attach database (SQLite)

**`ATTACH DATABASE 'path' AS alias`** opens **another** file as a **schema**. If injection can influence **`path`**, attackers may **read** or **merge** data across **trust** boundaries.

## Context

`ATTACH 'c:/windows/temp/x.db' AS p` merges another DB into the session—read/write across trust boundaries if paths are injectable.
## Technique

Use to pull secrets from a second on-disk DB the app should not touch.
## Practice

- Path traversal variants: `../` segments when concatenation is naive.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Remote code execution (SQLite)](index.md)
