---
title: "GROUP BY duplicate entry errors for MySQL error-based SQL injection and information leaks"
description: Duplicate-entry error channels that embed concatenated subquery output on certain MySQL versions (double-query technique).
keywords:
  - GROUP BY SQL injection
  - duplicate entry
  - MySQL
---

# GROUP BY errors

## Context

The `GROUP BY` + `floor(rand(0)*2)` pattern (sometimes called “double query”) relied on duplicate-key error text including material from an inner `CONCAT` subquery. Behavior varies by MySQL version and `sql_mode`. Treat as a **legacy / CTF** pattern unless you confirm it on the target version in a lab.

## Theory

Fragile across patches; prefer `UPDATEXML` / `EXTRACTVALUE` error channels when the expression context allows. Keep this technique documented for older stacks and courseware.

## Practice

- Reproduce only on pinned MySQL images used in your exercise. Compare error text before and after minor version upgrades.

## Tools

- **sqlmap**
- **Burp Suite**
