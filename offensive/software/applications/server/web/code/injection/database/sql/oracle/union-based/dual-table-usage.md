---
title: "Oracle DUAL table in UNION probes and subqueries"
description: The DUAL dummy table for expression evaluation in Oracle—common in union probe payloads.
keywords:
  - FROM dual
  - Oracle DUAL
---
# Dual table usage (Oracle)

**`DUAL`** is Oracle’s **one-row** helper table for **`SELECT`** expressions without real tables, often used in **probe** and **union** tests.

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
