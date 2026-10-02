---
title: "IBM Db2 boolean-based blind SQL injection: CASE and string functions"
description: Inference using Db2 string and conditional functions, SUBSTR, LENGTH, and EXISTS-style tests.
keywords:
  - CASE WHEN
  - SUBSTR
  - Db2
---
# Boolean-based (IBM Db2)

**Boolean-based** blind SQLi compares **substring** tests with **`CASE`**, **`SUBSTR`**, **`LENGTH`**, and related **predicates** on **`SYSIBM.SYSDUMMY1`** or injected **WHERE** clauses.

## Context

Blind SQLi infers bytes or tokens when the app returns the same shape for true/false branches, no row data, no SQL text in errors.
## Technique

Drive **boolean** conditions (`AND`, `OR`, `CASE`) so one branch matches application logic (200 vs empty, price change, etc.). Slice secrets with substring/length primitives; switch to **time** channels when boolean oracles are noisy.
## Practice

- Confirm an oracle: fixed request baseline, then flip `AND 1=1` vs `AND 1=2` (or dialect equivalents).
- Binary-search or iterate charset per position; throttle to avoid WAF/rate limits.
- Log response metadata (length, hash of body, timing) when visual diff is weak.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
