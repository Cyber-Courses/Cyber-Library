---
title: "Oracle credential extraction via UNION: all_users and data dictionary"
description: High-sensitivity extraction paths, password hashes and account metadata when overly broad SELECT exists.
keywords:
  - all_users
  - sys.user$
---
# Credential extraction (Oracle)

**Data dictionary** views may expose **hashes** or **metadata** depending on version and grants. **Application** accounts should not **read** credential stores broadly.

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
