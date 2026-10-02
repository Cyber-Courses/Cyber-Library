---
title: "Inferring column names without information_schema in MySQL UNION SQL injection"
description: Fallback techniques when information_schema is unavailable or filtered, subqueries, error channels, and brute patterns in MySQL UNION SQLi.
keywords:
  - MySQL SQL injection
  - no information_schema
  - blind SQLi
---

# Column names without metadata

## Context

Some deployments revoke **`SELECT`** on **`information_schema`**, use **views** that hide metadata, or a WAF blocks the literal string. You still need column names to target interesting fields. This page lists **recognition** and **lab** approaches; production testing requires explicit scope.

## Theory

Alternatives include: **error-based** channels that leak substrings from illegal casts or XPath-style misuse on MySQL where applicable; **boolean** tests that compare substring of `(SELECT column_name FROM …)` against known guesses; **`LIMIT` offset,1** walks over unknown column sets in subqueries that only succeed when a column exists; and **file** / **log** channels if `INTO OUTFILE` or `general_log` abuse is available (environment-specific and often disabled).

## Practice

### Guess-and-confirm on a known table

- If `users` is known from error text or source, binary-search column names with boolean conditions:

```sql
AND EXISTS(SELECT x FROM users WHERE x.username IS NOT NULL)
```

Replace the guessed column name until the truthiness of the page toggles (in blind boolean setups).

### Partial-name enumeration

- Use substring comparisons on `(SELECT password FROM users LIMIT 0,1)` without ever naming `password` in the outer union if the point is data exfiltration once one column is known from context.

## Tools

- **sqlmap**
- **Burp Suite**
