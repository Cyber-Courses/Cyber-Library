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

SQL keywords are split with inline comments `/**/` (which replace whitespace) and with case variation, not with concatenation: `||`/`CONCAT` build string values inside expressions, and the result is not reparsed as a keyword, so they cannot reconstruct `UNION` or `SELECT`. Use comments and case for keywords:

```sql
' UnIoN/**/SeLeCt/**/TABNAME/**/FROM/**/SYSCAT.TABLES-- 
```

Concatenation instead defeats filters on string literal values (a table name, a payload string), reconstructing a blocked substring from pieces with `CONCAT`, `CHR()`, `TRANSLATE`, or `REPLACE`. The special registers (`CURRENT USER`, `CURRENT SERVER`) also provide data without function-call syntax a filter might flag.

As with the other engines, the goal is to express the same query through synonyms and encodings the filter does not recognize rather than to defeat it head-on, combining several of these where a filter blocks more than one pattern. Note that a Db2 hex literal (`x'...'`) still uses quotes, so for genuinely quote-free construction rely on `CHR()` concatenation.

## Tools

- **sqlmap**: tamper scripts automate `CHR()`, comment, and case rewrites.
- **ghauri**: built-in WAF evasion for Db2 payloads.
- **Burp Repeater**: hand-tune encodings and special registers until the filter is bypassed.

## References

- IBM Db2 SQL Reference: CHR, CONCAT, CAST, comments, special registers
- OWASP Testing Guide: Testing for SQL Injection
