---
title: "Boolean-based blind SQL injection in Oracle"
description: "Inferring Oracle data one character at a time from true/false response differences using SUBSTR, ASCII, and LENGTH over DUAL."
keywords:
  - boolean based blind
  - blind SQL injection
  - SUBSTR ASCII
  - Oracle inference
  - DUAL
---

# Blind

When the response reflects neither rows nor error text, a condition whose truth depends on the data still leaks it through the behavior. If a true and a false condition render differently, each injected comparison answers one yes/no question.

Oracle isolates a character with `SUBSTR()` and converts it with `ASCII()` for comparison. Subqueries select `FROM DUAL` or a target table:

```sql
' AND ASCII(SUBSTR((SELECT user FROM dual),1,1))>77-- 
```

A binary search pins each character in about seven requests, then the position advances. Find the length first with `LENGTH()`:

```sql
' AND LENGTH((SELECT user FROM dual))=5-- 
```

Confirm the oracle with a known true/false pair (`' AND 1=1--` versus `' AND 1=2--`). Reading from a multi-row source needs a single-row selector, since Oracle has no `LIMIT`:

```sql
' AND ASCII(SUBSTR((SELECT password FROM sys.user$ WHERE ROWNUM=1),1,1))>64-- 
```

Because this generates many near-identical requests it is almost always automated, but the single-request comparison is the primitive that lets you adapt when a tool stalls, and it falls back to time-based inference when no boolean difference is visible.

## Tools

- **sqlmap**: automated boolean-based extraction (`--technique=B`).
- **ghauri**: fast boolean inference with strong WAF evasion.
- **Burp Intruder**: scripts the per-character comparison requests by hand.

## References

- Oracle Database SQL Language Reference: SUBSTR, ASCII, LENGTH, ROWNUM
- PortSwigger Web Security Academy: Blind SQL injection
