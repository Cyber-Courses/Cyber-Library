---
title: "Time-based blind SQL injection in MSSQL"
description: "Inferring SQL Server data from conditional response delays with WAITFOR DELAY, and why it generally needs a statement (stacked) context."
keywords:
  - time based blind
  - WAITFOR DELAY
  - conditional delay
  - MSSQL timing
---

# Time-based

When true and false look identical, timing carries the signal. SQL Server delays with `WAITFOR DELAY '0:0:5'`, which pauses for the given time.

`WAITFOR` is a statement, not an expression, so it runs where a statement is allowed: a stacked query, or an injection point that is itself a full statement. An unconditional delay confirms the channel:

```sql
'; WAITFOR DELAY '0:0:5'-- 
```

Make it conditional with `IF`, so the delay fires only on a true test:

```sql
'; IF (ASCII(SUBSTRING((SELECT TOP 1 name FROM sys.sql_logins),1,1))>77) WAITFOR DELAY '0:0:5'-- 
```

A slow response means the character's code point is above 77; a prompt response means it is not. Binary-search each position exactly as in boolean extraction, reading latency instead of content.

Where stacked queries are not available, `WAITFOR` cannot be injected as its own statement. The fallback is a heavy expression whose cost is conditional, for example forcing a large cross join only on a true branch, though this is far less precise than `WAITFOR`. Because timing is sensitive to load and jitter, keep delays several seconds long and repeat a positive hit before trusting it.

## Tools

- **sqlmap**: automated time-based blind extraction with `WAITFOR DELAY`.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- Microsoft SQL Server Documentation: WAITFOR, IF, control-of-flow
- PortSwigger Web Security Academy: Blind SQL injection
