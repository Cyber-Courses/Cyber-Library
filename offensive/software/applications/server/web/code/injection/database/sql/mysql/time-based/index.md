---
title: "Time-based blind SQL injection (MySQL): SLEEP, conditional delays, and subselect timing"
description: Inferring data through request latency using SLEEP, BENCHMARK, or heavy subselects in conditional expressions.
keywords:
  - time-based SQLi
  - SLEEP
  - MySQL
---

# Time-based SQLi

When **boolean** **oracles** are **noisy** or **missing**, **delay** **injection** uses **`SLEEP(n)`** or **`BENCHMARK`** inside a **branch** so **true** **predicates** **wait** **longer** **than** **false**. **Network** **jitter** requires **multiple** **samples** **per** **guess** in **real** **tests**.

## Pages

- [Using conditional statements](using-conditional-statements.md)
- [Using SLEEP in a subselect](using-sleep-in-a-subselect.md)
