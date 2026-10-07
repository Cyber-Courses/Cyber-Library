---
title: "Time-based blind SQL injection in PostgreSQL"
order: 8
description: "Inferring PostgreSQL data from conditional response delays with pg_sleep driven by a CASE expression, when true and false look identical."
keywords:
  - time based blind
  - pg_sleep
  - conditional delay
  - CASE WHEN
  - PostgreSQL timing
---

# Time-based

When true and false produce the same response, timing carries the signal: a payload that delays only on a true condition turns response latency into the yes/no answer. PostgreSQL delays with `pg_sleep(seconds)`.

An unconditional delay confirms the technique is live. `pg_sleep` is a set-returning function, so it is used from a `FROM` clause:

```sql
' AND 1=(SELECT 1 FROM pg_sleep(5))-- 
```

Make it conditional by gating the sleep with `CASE`, so the delay fires only when the test is true:

```sql
' AND 1=(CASE WHEN (ascii(substring(current_user,1,1))>77) THEN (SELECT 1 FROM pg_sleep(5)) ELSE 1 END)-- 
```

Both branches of the `CASE` return 1, so the surrounding `1=...` is always true and the response is unchanged except for the delay, which only occurs on the true branch. Binary-search each character exactly as in boolean extraction, reading latency instead of content.

Where the injection permits stacked queries, a second statement with a guarded `pg_sleep` is an alternative:

```sql
'; SELECT CASE WHEN (ascii(substring(current_user,1,1))>77) THEN pg_sleep(5) ELSE pg_sleep(0) END-- 
```

Timing is sensitive to network jitter and load, so keep delays several seconds long and repeat a positive hit before trusting it.

## Tools

- **sqlmap**: automated time-based blind extraction with `pg_sleep`.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- PostgreSQL Documentation: `pg_sleep`, CASE, set-returning functions
- PortSwigger Web Security Academy: Blind SQL injection
