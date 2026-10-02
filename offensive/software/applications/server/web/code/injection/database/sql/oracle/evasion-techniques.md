---
title: "Filter and WAF evasion for Oracle injection"
description: "Evading filters in Oracle injection with inline comments, CHR() string building, concatenation, and case manipulation."
keywords:
  - WAF bypass
  - CHR function
  - inline comments
  - concatenation
  - Oracle filter evasion
---

# Evasion techniques

Oracle's syntax offers several ways past signature filters that block quotes or keywords.

Strings are built without quotes from character codes with `CHR()` joined by `||`, which defeats quote filters and quoted-string signatures:

```sql
-- 'SCOTT' without quotes
CHR(83)||CHR(67)||CHR(79)||CHR(84)||CHR(84)
```

Inline comments `/**/` replace whitespace, breaking signatures that match space-separated tokens, and concatenation splits a keyword that a filter looks for as one string literal:

```sql
' UNION/**/SELECT/**/banner/**/FROM/**/v$version-- 
```

Case is free to vary since Oracle keywords are case-insensitive (`UnIoN SeLeCt`), which bypasses naive case-sensitive blocklists. Numbers can be written in alternative forms, and values can be reconstructed from `CHR`, `CONCAT`, or `UTL_RAW`/`TO_CHAR` conversions to avoid filtered substrings.

`CASE WHEN`/`DECODE` restructure a condition so it no longer matches a pattern while keeping the same logic, which helps when a specific comparison operator or keyword is filtered. As with the other engines, the goal is to express the same query through synonyms and encodings the filter does not recognize, rather than to defeat the filter head-on, and these combine freely since a filter usually blocks several patterns at once.

## Tools

- **sqlmap**: tamper scripts (for example `space2comment`, `charencode`) automate these rewrites.
- **ghauri**: built-in WAF evasion for Oracle payloads.
- **Burp Repeater**: hand-tune comments, `CHR()`, and case until the filter is bypassed.

## References

- Oracle Database SQL Language Reference: CHR, comments, expressions
- OWASP Testing Guide: Testing for SQL Injection
