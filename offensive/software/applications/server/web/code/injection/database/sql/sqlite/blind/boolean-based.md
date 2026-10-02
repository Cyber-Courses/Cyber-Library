---
title: "SQLite boolean-based blind SQL injection: substr, hex, and CASE"
description: Inference without verbose errors using SQLite string and conditional primitives.
keywords:
  - substr
  - unicode
  - CASE WHEN
---
# Boolean-based (SQLite)

**Boolean-based** blind SQLi compares **substring** tests and **length** checks, often with **`substr`**, **`hex`**, **`unicode`**, and **`CASE`**.

## Context

SQLite blind uses `substr`, `hex`, `unicode`, and `CASE` when errors and union output are suppressed.
## Technique

Boolean oracle: same row count / same JSON shape for true vs false injected predicates.
## Practice

- Iterate charset with `LIKE` prefix tests or numeric comparisons on `hex(secret)`.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
