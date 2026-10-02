---
title: "IBM Db2 error-based SQL injection: XML helpers, casts, and SQLSTATE"
description: Error-driven extraction in Db2—invalid casts, XML functions, and SIGNAL patterns that echo controlled expressions.
keywords:
  - SQLSTATE
  - XMLAGG
  - Db2 error-based
---
# Error-based (IBM Db2)

**Error-based** channels use **forced** **type** errors, **XML** parsing failures, or **`SIGNAL`**-style exceptions that include **attacker-influenced** text in **messages** returned to the client.

## Context

Db2 error channels: forced casts, `XML*` helpers, `SIGNAL`, divide-by-zero—mirror Oracle-style error SQLi.
## Technique

Embed scalar subqueries in expressions that surface in `SQLCODE`/`SQLSTATE` text returned to the client.
## Practice

- Chunk with `SUBSTR` if messages truncate.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [IBM Db2 (SQLi)](index.md)
