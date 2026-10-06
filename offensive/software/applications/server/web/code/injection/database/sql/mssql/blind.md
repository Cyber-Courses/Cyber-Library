---
title: "Boolean-based blind SQL injection in MSSQL"
order: 1
description: "Inferring SQL Server data one character at a time from true/false response differences using SUBSTRING, ASCII, and LEN."
keywords:
  - boolean based blind
  - blind SQL injection
  - SUBSTRING ASCII
  - MSSQL inference
  - binary search
---

# Blind

When the response reflects neither rows nor error text, a condition whose truth depends on the data still leaks it through the behavior. If a true and a false condition render differently, each injected comparison answers one yes/no question.

SQL Server isolates a character with `SUBSTRING()` and converts it with `ASCII()` (or `UNICODE()` for Unicode) for comparison:

```sql
' AND ASCII(SUBSTRING((SELECT TOP 1 name FROM sys.sql_logins),1,1))>77-- 
```

A binary search pins each character in about seven requests, then the position advances. Find the length first with `LEN()`:

```sql
' AND LEN((SELECT TOP 1 name FROM sys.sql_logins))=4-- 
```

Confirm the oracle with a known true/false pair (`' AND 1=1--` versus `' AND 1=2--`). `TOP 1` selects a single row because SQL Server has no `LIMIT`, and `ORDER BY` with `OFFSET ... FETCH` walks successive rows when more than the first is needed.

Because this generates many near-identical requests it is almost always automated, but the single-request comparison is the primitive that lets you adapt when a tool stalls, and it falls back cleanly to time-based inference when no boolean difference is visible.

## Tools

- **sqlmap**: automated boolean-based blind extraction with binary search.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- Microsoft SQL Server Documentation: SUBSTRING, ASCII, LEN, TOP
- PortSwigger Web Security Academy: Blind SQL injection
