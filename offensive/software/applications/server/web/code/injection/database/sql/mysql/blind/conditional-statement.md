---
title: "IF and CASE predicates in blind MySQL SQL injection for boolean inference"
description: Using IF and CASE expressions in injected SQL to branch on true or false and leak data through a boolean response channel.
keywords:
  - IF SQL injection
  - blind SQLi
  - MySQL
---

# Conditional predicates

## Context

When the injectable query fragment can be shaped as a **predicate** (inside `WHERE` or as an operand), you can wrap a **secret** comparison in `IF(condition, true_expr, false_expr)` so the **overall** query **succeeds** or **fails** in ways the page reflects. Use only in **authorized** environments.

## Theory

`IF((SELECT SUBSTRING(password,1,1) FROM users LIMIT 1)='a',1,0)` inside a numeric context can flip a **cart** total, **row** **count**, or **sort** **order** if the app prints different HTML for zero vs nonzero results. `CASE WHEN ... THEN ... ELSE ... END` is equivalent. The **oracle** is whatever observable differs between the two branches (length, keyword presence, timing if one branch is heavier).

## Practice

### Single-bit test

- Fix a baseline true request. Inject `AND IF(1=1,1,0)` and `AND IF(1=2,1,0)` to confirm the **oracle** moves. Then replace `1=1` with a substring predicate on secret data.

### Binary search on ASCII

- For byte `B`, test `B>63`, then narrow, or test bits with `ASCII(SUBSTRING(...)) & 1`, `&2`, … depending on noise tolerance.

## Tools

- **Burp Suite**
- **sqlmap**
