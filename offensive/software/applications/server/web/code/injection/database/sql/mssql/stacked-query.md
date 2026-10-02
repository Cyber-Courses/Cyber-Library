---
title: "MSSQL stacked queries in SQL injection: semicolons, batches, and driver behavior"
description: How multiple T-SQL batches interact with SQL injection—semicolon chaining, EXEC, and client support for multiple statements.
keywords:
  - stacked query
  - sp_executesql
  - MSSQL SQL injection
---
# Stacked query (MSSQL)

**Stacked** or **batched** execution means submitting **more than one statement** in a single request (for example after a `;`). Whether this works depends on the **API and driver** (some client stacks disallow multiple statements), not only the database.

## Context

Stacked batches separate statements with `;` — works only if the driver/API allows multiple statements in one call.
## Technique

After confirmation, chain `EXEC`, `sp_executesql`, or DDL/DML in sequence; some ORMs strip semicolons—test raw HTTP.
## Practice

- Probe with `;SELECT 1--` vs error; watch ORM parameter binding.
- If only one statement is allowed, pivot to union/blind instead of stacking.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Command execution](command-execution.md)
