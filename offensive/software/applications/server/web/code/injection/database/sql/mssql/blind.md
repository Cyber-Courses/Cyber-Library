---
title: "MSSQL blind SQL injection: substring and Unicode inference without verbose errors"
description: Boolean and content inference in Microsoft SQL Server when errors are suppressed—SUBSTRING, ASCII, and related T-SQL primitives.
keywords:
  - MSSQL blind SQLi
  - SUBSTRING
  - ASCII
  - CHARINDEX
---
# Blind (MSSQL)

When the application hides database errors, attackers still infer secrets by **branching on true/false conditions** and comparing substrings of metadata or data. T-SQL offers `SUBSTRING`, `LEFT`, `RIGHT`, `CHARINDEX`, `ASCII`, and `UNICODE` for character-by-character extraction.

## Context

SQL Server hides errors but still evaluates your injected predicate; T-SQL gives `SUBSTRING`, `ASCII`, `UNICODE`, `CHARINDEX` for bitwise extraction.
## Technique

Build `AND ASCII(SUBSTRING(@@version,1,1))>N` style tests, or compare hashes of substrings if you only have row-existence oracles.
## Practice

- Use `SUBSTRING`/`LEFT`/`RIGHT` with position loops; `UNICODE` for wide chars.
- If stacked queries work, you can sometimes materialize intermediates into a temp table for cleaner extraction.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Time-based](time-based.md)
