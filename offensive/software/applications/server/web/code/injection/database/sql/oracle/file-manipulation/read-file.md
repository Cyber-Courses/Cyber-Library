---
title: "Oracle read file via SQL: UTL_FILE and BFILE directory objects"
description: Reading host files through Oracle directory mappings, privilege and OS path controls.
keywords:
  - UTL_FILE.FOPEN
  - BFILE
---
# Read file (Oracle)

**UTL_FILE** and **`BFILE`** reads depend on **`DIRECTORY`** objects pointing at **filesystem** paths the Oracle process can read.

## Context

`UTL_FILE.FOPEN`/`GET_LINE` reads through Oracle directory mappings the service account can access.
## Technique

Pick paths like listener logs, wallet files, or app config, depends on directory grants.
## Practice

- If `UTL_FILE` is locked down, try LOB loaders or external tables per version.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
