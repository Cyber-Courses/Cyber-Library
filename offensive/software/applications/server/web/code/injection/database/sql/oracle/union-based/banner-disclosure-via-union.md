---
title: "Oracle banner disclosure via UNION: v$version and gv$version"
description: Version fingerprinting through union-visible views when injection can select from dictionary or dynamic performance views.
keywords:
  - v$version
  - banner
  - Oracle union SQLi
---
# Banner disclosure via UNION (Oracle)

**`v$version`** / **`gv$version`** expose **banner** strings useful for **patch** planning. Union-based injection may surface this when the vulnerable query can **project** arbitrary expressions.

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

## See also

- [Union-based (Oracle)](index.md)
