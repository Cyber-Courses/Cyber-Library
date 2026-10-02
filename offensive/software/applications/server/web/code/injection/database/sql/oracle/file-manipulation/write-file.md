---
title: "Oracle write file via SQL: UTL_FILE and LOB write primitives"
description: File creation through Oracle packages, relevant to webshell-style drops when the process can write web roots (operational layout dependent).
keywords:
  - UTL_FILE.PUT_LINE
  - DBMS_LOB.WRITE
---
# Write file (Oracle)

**Write** access via **`UTL_FILE.PUT_LINE`** or **`DBMS_LOB`** requires **`DIRECTORY`** write privileges and OS permissions for the **Oracle software owner**.

## Context

`UTL_FILE.PUT_LINE` / `DBMS_LOB.WRITE` drops content to mapped directories, webshell only if web tier shares a writable path.
## Technique

Combine with `DBMS_SCHEDULER` for execute-after-write if policy allows.
## Practice

- Validate web root layout before guessing paths.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
