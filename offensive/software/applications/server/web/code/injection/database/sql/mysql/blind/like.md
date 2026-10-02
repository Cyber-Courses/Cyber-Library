---
title: "LIKE-based blind SQL injection (MySQL) for character-by-character inference"
description: Using LIKE wildcards and prefix tests in boolean blind SQLi to infer characters without printing them.
keywords:
  - LIKE SQL injection
  - blind SQLi
  - MySQL
---

# LIKE channel

## Context

`LIKE` supports `%` and `_` wildcards. In blind SQLi you can test whether a secret string **starts with** a guess: `column LIKE 'adm%'` is true or false in one request. Combined with **binary search** on the next character, you reduce requests versus naive per-position 256-way brute force.

## Theory

`LIKE BINARY` can enforce case sensitivity when the column collation is loose. `SUBSTRING` slices one position at a time: `LIKE CONCAT(SUBSTRING(pass,1,1),'%')` patterns. For **hex** or **binary** data, use `HEX()` and compare prefix of hex digits.

## Practice

### Prefix confirmation

- After confirming `username='admin'` exists, test password prefix: `AND (SELECT password FROM users WHERE id=1) LIKE 'a%'` iterate `a`-`z`, `0`-`9`, symbols per policy.

### Wildcard pitfalls

- `%` and `_` in user-controlled **data** can **match** too much; in **injection** you control the **pattern** string, escape literals if the app adds backslashes.

## Tools

- **Burp Suite Intruder**
- **sqlmap**
