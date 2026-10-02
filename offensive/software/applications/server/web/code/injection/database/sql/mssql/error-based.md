---
title: "MSSQL error-based SQL injection: CONVERT, CAST, and type mismatch channels"
description: Error-based data extraction in SQL Server when type conversion or constraint violations surface query fragments in error messages.
keywords:
  - CONVERT
  - CAST
  - MSSQL error-based SQLi
---
# Error-based (MSSQL)

When the database returns **detailed errors** to the client, failed **conversions** (`CONVERT`, `CAST`) and similar operations can leak **substrings** of attacker-selected expressions embedded in the message text.

## Context

Verbose SQL Server errors echo conversion failures, embed subqueries inside `CONVERT`/`CAST` targets so the message leaks scalar data.
## Technique

Force type mismatches or out-of-range conversions where the error text includes your expression result.
## Practice

- Enumerate column types from union probes first to pick convertible sinks.
- Truncate messages may require chunking via substring in nested converts.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
