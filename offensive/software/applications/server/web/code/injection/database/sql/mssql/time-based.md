---
title: "MSSQL time-based SQL injection: WAITFOR DELAY and conditional timing"
description: Inferring data through deliberate delays in SQL Server—WAITFOR and conditional branches when blind inference is possible.
keywords:
  - WAITFOR DELAY
  - time-based SQLi
  - MSSQL
---
# Time-based (MSSQL)

**Time-based** inference uses **`WAITFOR DELAY`** (and conditional wrappers) so that **true** versus **false** conditions produce measurably different response times. It is a blind technique when errors and union channels are unavailable.

## Context

`WAITFOR DELAY '0:0:5'` is the canonical pause; stack it behind `IF`/`CASE` when you control a full statement.
## Technique

Conditional delay: `IF (condition) WAITFOR DELAY ...` — measure p50/p95 response time over several runs.
## Practice

- Escalate delay duration if the app server buffers or pools connections oddly.
- Pair with DNS/OOB if outbound is allowed and blind is too slow.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Blind](blind.md)
