---
title: "Boolean-based blind SQL injection in SQLite"
order: 1
description: "Inferring SQLite data one character at a time from true/false response differences using substr and unicode over sqlite_master and tables."
keywords:
  - boolean based blind
  - blind SQL injection
  - substr unicode
  - SQLite inference
  - sqlite_master
---

# Blind

Because SQLite's error channel is weak, boolean inference is the dependable way to read data when rows are not reflected. If a true and a false condition render differently, each injected comparison answers one yes/no question.

SQLite isolates a character with `substr()` and converts it with `unicode()` (its `ASCII`-equivalent) for comparison:

```sql
' AND unicode(substr((SELECT password FROM users LIMIT 1),1,1))>77-- 
```

A binary search pins each character in about seven requests, then the position advances. Find the length first with `length()`:

```sql
' AND length((SELECT password FROM users LIMIT 1))=32-- 
```

Confirm the oracle with a known true/false pair (`' AND 1=1--` versus `' AND 1=2--`). SQLite supports `LIKE` for a compact oracle that needs no `substr`, anchoring one character at a time:

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'-- 
```

`LIKE` is case-insensitive for ASCII by default; use `GLOB` (`GLOB 'a*'`) for a case-sensitive match. The schema is read the same way from `sqlite_master`. Blind extraction is slow and usually automated, but the single-request comparison is what lets you adapt when a tool stalls, and it is the primary channel for SQLite given the limited error surface.

## Tools

- **sqlmap**: automated boolean-based extraction (`--technique=B`).
- **ghauri**: fast boolean inference with strong WAF evasion.
- **Burp Intruder**: scripts the per-character comparison requests by hand.

## References

- SQLite Documentation: substr, unicode, length, LIKE, GLOB
- PortSwigger Web Security Academy: Blind SQL injection
