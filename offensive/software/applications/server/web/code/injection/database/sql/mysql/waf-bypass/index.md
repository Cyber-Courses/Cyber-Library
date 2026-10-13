---
title: "WAF and filter bypass for MySQL injection"
order: 11
description: "Reaching MySQL keywords, functions, and catalog tables past signature filters using versioned comments, encoding, alternative objects, and charset tricks."
keywords:
  - WAF bypass
  - MySQL filter evasion
  - versioned comments
  - wide byte injection
  - information_schema alternative
---

# WAF bypass

Web application firewalls and input filters block injection by matching signatures: keywords like `UNION SELECT`, function names, and the string `information_schema`. MySQL's dialect offers enough equivalents that most single-signature filters can be worked around, which is why keyword blocklists are a weak defense.

The main families are: comment tricks that split or conditionally run keywords, encodings that remove quotes and spaces from the payload, alternative objects that reach the same data under a different name, and charset tricks that neutralize escaping. They are combined as needed, since a filter usually blocks several patterns at once.

## Pages

- **[Versioned comments](versioned-comments.md)**: `/*! */` and `/**/` to split and conditionally execute keywords.
- **[Wide-byte injection](wide-byte-gbk.md)**: GBK multibyte sequences that swallow escaping backslashes.
- **[Alternative catalog and functions](alternative-catalog.md)**: reach schema and version without the blocked names.

## Tools

- **sqlmap**: extensive tamper scripts for filter and WAF evasion.
- **Burp Repeater**: craft and iterate bypass payloads manually.

## References

- MySQL Reference Manual: comment syntax, character sets
- OWASP Testing Guide: Testing for SQL Injection
