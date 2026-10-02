---
title: "SQLite time-based blind SQL injection: heavy queries and delay emulation"
description: Timing channels in SQLite without a dedicated SLEEP, expensive operations and resource contention as weak signals.
keywords:
  - randomblob
  - SQLite timing
---
# Time-based (SQLite)

SQLite lacks a standard **`SLEEP`**. Attackers may emulate **delays** with **expensive** expressions (**`randomblob`**, large **`LIKE`** scans) when **conditional** branches can choose **costly** versus **cheap** plans, **noisier** than server RDBMS delays.

## Context

SQLite has no `sleep`; emulate delay with **`randomblob(N)`**, huge `LIKE` scans, or recursive CTE burn, noisy and environment-dependent.
## Technique

Measure timing statistically; increase work units until signal clears jitter.
## Practice

- Prefer blind boolean when the oracle is stable, faster than timing.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
