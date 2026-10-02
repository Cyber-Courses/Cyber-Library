---
title: "Oracle UNION data type matching: TO_CHAR, CAST, and ORA-01790"
description: Aligning column types in UNION, CAST and TO_CHAR to satisfy all branches and avoid type mismatch errors.
keywords:
  - ORA-01790
  - TO_CHAR
  - CAST
---
# Data type matching (Oracle)

**UNION** requires **compatible** types per column. **`TO_CHAR`**, **`CAST`**, and **`NULL`** help align **string**, **number**, and **date** columns.

## Context

Oracle union steps: count columns, align types, then project data from `v$version`, `all_users`, or app tables.
## Technique

Use `NULL` placeholders, `TO_CHAR` casts, and `ORDER BY` index probing.
## Practice

- Row limit: `WHERE ROWNUM=1` or wrapped subqueries for ordered top-N.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
