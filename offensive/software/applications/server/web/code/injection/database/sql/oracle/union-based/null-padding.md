---
title: "Oracle UNION null padding for column alignment"
description: Using NULL placeholders while mapping column positions in union-based SQL injection testing.
keywords:
  - UNION SELECT NULL
---
# Null padding (Oracle)

**`NULL`** pads unknown columns during **union** **mapping** because **`NULL`** is assignable in many type contexts.

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
