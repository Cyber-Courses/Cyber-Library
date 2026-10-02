---
title: "Boolean-based blind SQL injection in MySQL"
description: "Inferring MySQL data one character at a time from true/false differences in the response when no rows or errors are reflected."
keywords:
  - boolean based blind
  - blind SQL injection
  - MySQL inference
  - SUBSTRING ASCII
  - binary search extraction
---

# Boolean blind

When the application reflects neither rows nor error text, injection still leaks data through its behavior. If a true condition and a false condition produce observably different responses (a different page, length, status, or message), each injected comparison answers one yes/no question, and enough questions reconstruct any value.

The building block is a comparison whose truth depends on a single character of the target. `SUBSTRING` isolates the character and `ASCII` turns it into a number to compare:

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>77-- 
```

A binary search finds each character in about seven requests: halve the range on every answer until the exact code point is known, then move to position two, three, and so on. The current database, user, and schema names are read the same way, swapping the inner subquery.

Blind extraction is slow but reliable, and it is the fallback whenever richer channels (union, error, out-of-band) are closed. The pages below cover the standard character oracle and how to keep it working when common characters are filtered.

## Pages

- **[Boolean extraction](boolean-based.md)**: the `SUBSTRING`/`ASCII` character oracle with binary search.
- **[Comma-free extraction](comma-free-extraction.md)**: `LIKE` and `SUBSTRING ... FROM ... FOR` when commas are blocked.

## References

- MySQL Reference Manual: SUBSTRING, ASCII, string comparison
- PortSwigger Web Security Academy: Blind SQL injection
