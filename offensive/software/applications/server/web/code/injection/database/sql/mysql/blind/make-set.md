---
title: "MAKE_SET and bitmap-style extraction in blind MySQL SQL injection"
description: Using MAKE_SET with bitwise tests to pack multiple boolean conditions or to extract bits from a value in MySQL blind injection.
keywords:
  - MAKE_SET
  - blind SQLi
  - MySQL
---

# MAKE_SET

## Context

`MAKE_SET(bits, str1, str2, ...)` returns a comma-separated list of strings where the **bit** is set in **bits**. In injection, `MAKE_SET` can turn **bit** **tests** on `ASCII(SUBSTRING(...))` into **distinct** **strings** that change **sort** **order** or **GROUP** **CONCAT** **output** shape, another **oracle** when **boolean** **AND** is **filtered**.

## Theory

Example pattern: `MAKE_SET((ASCII(SUBSTRING(pass,1,1))>>0)&1, 'a','b')` (conceptual) ties **bit** **positions** to **different** **substrings** visible in **error** or **ordering** if echoed. Often used with **`FIND_IN_SET`** and **reporting** **queries** that **sort** by a **derived** **column**.

## Practice

### Map one bit per request

- In a lab, verify whether `MAKE_SET` expressions appear in `ORDER BY` injectable points and whether the **sort** **order** of rows **changes** the **HTML** **layout** (weak oracle).

## Tools

- **Burp Suite**
- **sqlmap**
