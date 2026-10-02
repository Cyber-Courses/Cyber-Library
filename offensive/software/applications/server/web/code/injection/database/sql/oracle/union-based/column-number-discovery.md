---
title: "Oracle UNION column count discovery: ORDER BY probing"
description: Discovering column count for UNION injection using ORDER BY n in Oracle.
keywords:
  - ORDER BY
  - column count
---
# Column number discovery (Oracle)

Classic **column count** discovery uses **`ORDER BY 1`**, **`ORDER BY 2`**, … until errors stop, then **`UNION SELECT`** with matching arity.

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
