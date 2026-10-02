---
title: "Oracle row filtering with ROWNUM for UNION output"
description: Limiting rows in Oracle without LIMIT, ROWNUM in WHERE for top-N union results.
keywords:
  - ROWNUM
  - top-N
---
# Row filtering (Oracle)

**`ROWNUM`** filters **after** ordering in subquery patterns; use **`WHERE ROWNUM = 1`** or wrap ordered subqueries for **top-N** union output.

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
