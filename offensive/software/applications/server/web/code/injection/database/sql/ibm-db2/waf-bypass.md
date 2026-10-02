---
title: "WAF and filter bypass for IBM Db2 injection"
description: "Evading filters in IBM Db2 injection with CHR() and CONCAT string building, hex via CAST, inline comments, and case variation."
keywords:
  - WAF bypass
  - CHR function
  - CONCAT
  - hex literal
  - Db2 filter evasion
---

# WAF bypass

Db2's dialect offers the usual ways past signature filters that block quotes or keywords.

Strings are built without quotes from character codes with `CHR()` joined by `||` or `CONCAT()`, which defeats quote filters:

```sql
-- 'USERS' without quotes
CHR(85)||CHR(83)||CHR(69)||CHR(82)||CHR(83)
```

Hex can supply bytes through a cast, avoiding quoted literals in some positions:

```sql
CAST(x'5553455253' AS VARCHAR(5))
```

Inline comments `/**/` replace whitespace to break space-separated token signatures, and concatenation splits a keyword a filter matches as one string:

```sql
' UNION/**/SELECT/**/TABNAME/**/FROM/**/SYSCAT.TABLES-- 
```

Keywords are case-insensitive, so case variation (`UnIoN SeLeCt`) bypasses naive case-sensitive blocklists. Values can also be reconstructed with `CONCAT`, `TRANSLATE`, or `REPLACE` to avoid filtered substrings, and the special registers (`CURRENT USER`, `CURRENT SERVER`) provide data without function-call syntax a filter might flag.

As with the other engines, the goal is to express the same query through synonyms and encodings the filter does not recognize rather than to defeat it head-on, combining several of these where a filter blocks more than one pattern. Note that a Db2 hex literal (`x'...'`) still uses quotes, so for genuinely quote-free construction rely on `CHR()` concatenation.

## References

- IBM Db2 SQL Reference: CHR, CONCAT, CAST, comments, special registers
- OWASP Testing Guide: Testing for SQL Injection
