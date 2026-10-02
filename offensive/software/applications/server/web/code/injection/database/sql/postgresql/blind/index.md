---
title: "Boolean-based blind SQL injection in PostgreSQL"
description: "Inferring PostgreSQL data one character at a time from true/false differences in the response when no rows or errors are reflected."
keywords:
  - boolean based blind
  - blind SQL injection
  - PostgreSQL inference
  - substring ascii
  - binary search
---

# Boolean blind

When the application reflects neither rows nor error text, a condition whose truth depends on the target data still leaks it through the response. If a true and a false condition render differently, each injected comparison answers one yes/no question, and enough questions rebuild any value.

PostgreSQL isolates a character with `substring()` and converts it with `ascii()` for comparison:

```sql
' AND ascii(substring((SELECT passwd FROM pg_shadow LIMIT 1),1,1))>77-- 
```

A binary search pins each character in about seven requests, then the position advances. Confirm the oracle first with a known true and false pair:

```sql
' AND 1=1-- 
' AND 1=2-- 
```

Determine the length with `length()` so you know when to stop:

```sql
' AND length((SELECT passwd FROM pg_shadow LIMIT 1))=32-- 
```

The same oracle reads `current_user`, `current_database()`, and table data by swapping the inner subquery. Blind extraction is slow and almost always automated, but the single-request comparison is what lets you adapt when a tool stalls.

## Pages

- **[Boolean extraction](boolean-based.md)**: the `substring`/`ascii` character oracle with binary search.

## Tools

- **sqlmap**: automated boolean-based blind extraction.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- PostgreSQL Documentation: `substring`, `ascii`, `length`
- PortSwigger Web Security Academy: Blind SQL injection
