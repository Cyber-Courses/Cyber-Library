---
title: "Finding UNION column count in MySQL SQL injection with ORDER BY and NULL probes"
description: Finding the number of columns in the original SELECT so a UNION SELECT aligns without type errors in MySQL.
keywords:
  - ORDER BY column number
  - UNION SQL injection
  - column count
---

# Column count

## Context

Union-based SQL injection requires the injected `UNION SELECT` to project the **same number of columns** as the original query. MySQL returns explicit errors when counts or some type coercions mismatch. This page is for **authorized** labs and **in-scope** assessments only.

## Theory

Two classic probes: increment `ORDER BY n` until the application errors (MySQL accepts `ORDER BY` column **ordinal** in many query shapes), or inject `UNION SELECT NULL,NULL,...` with a growing list of `NULL` placeholders until the error disappears or the response stabilizes. `NULL` coerces across many column types for display in a result row.

## Practice

### ORDER BY ordinal probe

- In a reflected injection point that influences a `WHERE` clause but leaves the outer `SELECT` intact, append ` ORDER BY 1`, then `2`, `3`, … until the page errors or changes. The last successful ordinal estimates column count in simple stacks (verify against `UNION` because some queries wrap subselects).

### UNION NULL ladder

- Submit `UNION SELECT NULL` then `UNION SELECT NULL,NULL`, increasing until the error about **column count** disappears. Use only in a vulnerable local application or explicit written scope.

## Tools

- **Burp Suite**
- **sqlmap**
