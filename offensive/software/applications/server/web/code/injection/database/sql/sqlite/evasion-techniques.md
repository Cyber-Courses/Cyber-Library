---
title: "Filter and WAF evasion for SQLite injection"
description: "Evading filters in SQLite injection with inline comments, char() string building, and concatenation past keyword and quote blocklists."
keywords:
  - WAF bypass
  - char function
  - inline comments
  - concatenation
  - SQLite filter evasion
---

# Evasion techniques

SQLite's small but flexible syntax offers the usual ways around signature filters.

Strings are built without quotes from character codes with `char()` joined by `||`, which defeats quote filters:

```sql
-- 'users' without quotes
char(117)||char(115)||char(101)||char(114)||char(115)
```

Inline comments `/**/` replace whitespace to break space-separated token signatures, and concatenation splits a blocked keyword string:

```sql
' UNION/**/SELECT/**/name/**/FROM/**/sqlite_master-- 
```

Keywords are case-insensitive, so case variation (`UnIoN SeLeCt`) bypasses naive case-sensitive blocklists. Blob literals (`x'7573657273'`) help against keyword and string-value signatures, but note they still use single quotes, so unlike MySQL's unquoted `0x...` they do not defeat a quote filter; for genuinely quote-free construction use the `char()` form above. `CAST`/`unicode`/`char` conversions reconstruct filtered substrings.

Because SQLite is loosely typed and has a minimal parser, it also tolerates some oddities that confuse filters tuned for stricter engines, such as missing `FROM` clauses (`UNION SELECT 1,2`) and flexible literal forms. As with the other engines, the goal is to express the same query through synonyms and encodings the filter does not recognize rather than to defeat it head-on, combining several of these where a filter blocks more than one pattern.

## Tools

- **sqlmap**: tamper scripts (for example `space2comment`, `charencode`) automate these rewrites.
- **ghauri**: built-in WAF evasion for SQLite payloads.
- **Burp Repeater**: hand-tune comments, `char()`, and blob literals until the filter is bypassed.

## References

- SQLite Documentation: char, literals, comments, expressions
- OWASP Testing Guide: Testing for SQL Injection
