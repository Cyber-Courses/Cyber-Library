---
title: "REGEXP-based blind SQL injection (MySQL) for pattern-driven inference"
description: Using MySQL REGEXP for boolean tests on substrings and character classes in blind SQL injection.
keywords:
  - REGEXP SQL injection
  - blind SQLi
  - MySQL
---

# REGEXP channel

## Context

`REGEXP` and `RLIKE` are synonyms in MySQL. A predicate such as `(SELECT pass FROM users LIMIT 1) REGEXP '^a'` is **true** if the **secret** **starts** with **`a`**. Useful when `LIKE` is **filtered** but `REGEXP` is **not**, or when you need **character** **classes** (`[a-z]`, `[0-9]`).

## Theory

**Binary** sensitivity: `REGEXP BINARY '^A'` vs `'^a'`. For **Unicode** or **collation** quirks, compare **`HEX(prefix)`** with **`REGEXP '^61'`**-style **hex** **patterns** instead.

## Practice

### Anchor prefix test

- Inject `AND (SELECT column FROM t LIMIT 1) REGEXP '^expectedprefix'` and widen prefix character-by-character when the boolean oracle is reliable.

## Tools

- **Burp Suite**
- **sqlmap**
