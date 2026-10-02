---
title: "SLEEP in scalar subqueries for MySQL time-based blind SQL injection when IF is filtered"
description: Placing SLEEP in scalar subqueries when IF wrappers are filtered but subselect expressions are allowed.
keywords:
  - subselect SLEEP
  - time-based SQLi
  - MySQL
---

# SLEEP in subselect

## Context

Some filters block `IF` or comma-separated expression lists but still allow scalar subqueries. A pattern like `AND (SELECT SLEEP(5) FROM ... WHERE predicate)` delays only when the inner predicate is true. MySQL does not require `FROM DUAL` (unlike Oracle); use a valid subquery shape for the column context (e.g. `WHERE` on a derived row).

## Theory

Correlated subselects can tie delay to row matching, for example `EXISTS(SELECT 1 FROM users WHERE id=1 AND ASCII(SUBSTRING(pass,1,1))>64 AND SLEEP(5))`, subject to parser and type rules for the injection point.

## Practice

- When `AND IF` is blocked, try `AND EXISTS(SELECT SLEEP(5) WHERE <bit-test>)` or equivalent forms the backend accepts in a lab replica.

## Tools

- **sqlmap**
- **Burp Suite**
