---
title: "MySQL time-based injection with SLEEP"
description: "Conditional SLEEP payloads for MySQL blind SQL injection, using IF and correlated subqueries so the delay fires only when a test is true."
keywords:
  - SLEEP injection
  - conditional delay
  - IF SLEEP
  - MySQL time based extraction
---

# SLEEP

`SLEEP(n)` pauses for `n` seconds. Made conditional, it reports one bit: the response is slow when the test is true and fast when it is false.

The reliable inline form wraps the delay in `IF()`, which is valid anywhere an expression is allowed and needs no `FROM`:

```sql
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77,SLEEP(5),0)-- 
```

A five-second response means the character's code point is above 77; an immediate response means it is not. Binary-search each position exactly as in boolean extraction, reading latency instead of page content.

When the injection sits where a full subquery fits, a correlated `SELECT` over a real table with the delay in its `WHERE` is equally valid, because the `FROM` supplies the row context `SLEEP` needs:

```sql
' AND (SELECT 1 FROM users WHERE id=1 AND ASCII(SUBSTRING(password,1,1))>77 AND SLEEP(5))-- 
```

Avoid the common broken form `AND (SELECT SLEEP(5) WHERE <test>)`: MySQL allows a `SELECT` with no `FROM` only when it returns constants, and adding a `WHERE` without a `FROM` is a syntax error (unlike PostgreSQL and SQLite, which accept it), so this raises error 1064 rather than delaying. Keep the inner query to one row so `SLEEP` is evaluated predictably, and repeat any positive hit once to rule out a slow network.

## Tools

- **sqlmap**: automated time-based extraction using conditional SLEEP.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- MySQL Reference Manual: SLEEP, IF, subqueries
- PortSwigger Web Security Academy: Blind SQL injection
