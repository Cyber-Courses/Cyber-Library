---
title: "Oracle error-based SQL injection: ORA messages and type conversion"
description: Extracting data through Oracle error messages—XML helpers, type casts, and deliberate failures that echo expressions.
keywords:
  - ORA-
  - error-based SQLi
  - Oracle
---
# Error-based (Oracle)

Oracle often returns **`ORA-`** errors with **rich detail**. Attackers embed subqueries in contexts that **force conversion failures** or **XML** path errors so fragments of data appear in the message (for example **`CTXSYS`**, **`XMLType`**, **`UTL_INADDR`**-related errors depending on context).

## Context

Oracle `ORA-` messages often include substrings from failed conversions, XML paths, or `CTXSYS` helpers—embed scalar subqueries in those sinks.
## Technique

Pick functions that reflect query text in errors (`XMLType`, casting tricks, divide-by-zero with string numerator).
## Practice

- Chunk long strings: nested `SUBSTR` in repeated requests.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](index.md)
