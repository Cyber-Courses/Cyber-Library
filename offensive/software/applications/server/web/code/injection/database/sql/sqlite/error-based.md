---
title: "SQLite error-based SQL injection: type errors and divide-by-zero"
description: Leaking data through SQLite error messages when verbose errors reach the client—casts, division, and undefined functions.
keywords:
  - SQLite error-based
  - load_extension
---
# Error-based (SQLite)

SQLite can surface **type** errors, **division by zero**, and **undefined function** messages that echo **subexpressions** when **detailed** errors are enabled in the **application**.

## Context

SQLite returns detailed errors for bad casts, undefined functions, and divide-by-zero—embed subqueries in expressions that surface in the message.
## Technique

Use `CAST(x AS INT)` with string payloads, or `1/(CASE WHEN ... THEN 0 ELSE 1 END)` patterns.
## Practice

- If the app hides SQLite text, switch to blind.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [SQLite (SQLi)](index.md)
