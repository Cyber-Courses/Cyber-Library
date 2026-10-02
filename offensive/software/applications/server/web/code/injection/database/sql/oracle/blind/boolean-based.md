---
title: "Oracle boolean-based blind SQL injection: CASE, SUBSTR, and EXISTS"
description: Inference techniques on Oracle without verbose errors, conditional expressions against DUAL and metadata-free probing.
keywords:
  - CASE WHEN
  - SUBSTR
  - DUAL
  - Oracle blind SQLi
---
# Boolean-based (Oracle)

Attackers branch on **true/false** using **`CASE`**, **`EXISTS`**, **`INSTR`**, **`LENGTH`**, and **`SUBSTR`** against **`DUAL`** or injected predicates. Success is inferred from **page** or **API** behavior differences.

## Context

Oracle blind uses `CASE`, `SUBSTR`, `INSTR` against `DUAL` or your injected predicate to branch on single-bit truth.
## Technique

Compare response differences (HTTP size, JSON field presence) for `AND (SELECT CASE WHEN ... THEN 1 ELSE 0 END FROM dual)=1` patterns.
## Practice

- Use `LENGTH`/`SUBSTR` loops for secrets in `USER`/`ALL_*` views when union is blocked.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
