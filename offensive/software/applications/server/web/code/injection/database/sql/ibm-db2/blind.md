---
title: "Boolean-based blind SQL injection in IBM Db2"
order: 1
description: "Inferring IBM Db2 data one character at a time from true/false response differences using SUBSTR and ASCII over SYSIBM.SYSDUMMY1."
keywords:
  - boolean based blind
  - blind SQL injection
  - SUBSTR ASCII
  - Db2 inference
  - SYSIBM.SYSDUMMY1
---

# Blind

Because Db2's error channel is narrow, boolean inference is the dependable way to read data when rows are not reflected. If a true and a false condition render differently, each injected comparison answers one yes/no question.

Db2 isolates a character with `SUBSTR()` and converts it with `ASCII()` for comparison. Subqueries that return a scalar select `FROM SYSIBM.SYSDUMMY1` or a target table:

```sql
' AND ASCII(SUBSTR((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1),1,1))>77-- 
```

A binary search pins each character in about seven requests, then the position advances. Find the length first with `LENGTH()`:

```sql
' AND LENGTH((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1))=8-- 
```

Confirm the oracle with a known true/false pair (`' AND 1=1--` versus `' AND 1=2--`). Reading from a multi-row source needs a single-row selector, since Db2 has no `LIMIT`:

```sql
' AND ASCII(SUBSTR((SELECT password FROM users ORDER BY id FETCH FIRST 1 ROWS ONLY),1,1))>64-- 
```

Because this generates many near-identical requests it is almost always automated, but the single-request comparison is the primitive that lets you adapt when a tool stalls. It is the primary channel against Db2 given the limited error surface, with time-based inference as the fallback when no boolean difference is visible.

## Tools

- **sqlmap**: automated boolean-based extraction (`--technique=B`).
- **ghauri**: fast boolean inference with strong WAF evasion.
- **Burp Intruder**: scripts the per-character comparison requests by hand.

## References

- IBM Db2 SQL Reference: SUBSTR, ASCII, LENGTH, FETCH FIRST
- PortSwigger Web Security Academy: Blind SQL injection
