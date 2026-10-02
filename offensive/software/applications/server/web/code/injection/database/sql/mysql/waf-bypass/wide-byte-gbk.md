---
title: "Wide-byte (GBK) injection in MySQL"
description: "Bypassing charset-unaware escaping in MySQL by using GBK multibyte sequences so an added backslash is consumed and the quote survives."
keywords:
  - wide byte injection
  - GBK injection
  - addslashes bypass
  - multibyte charset
  - escaping bypass
---

# Wide-byte injection

Wide-byte injection defeats escaping that adds a backslash before a quote (`'` becomes `\'`) without accounting for the connection character set. On a multibyte charset such as GBK, a crafted lead byte combines with the escaping backslash to form one valid multibyte character, which leaves the following quote standing as a string delimiter.

Supplying `%bf%27` is the classic case. Naive escaping turns the `%27` quote into `%5c%27` (`\'`), producing the byte sequence `%bf%5c%27`. Under GBK, `%bf%5c` is a single valid character, so the backslash is consumed and the `%27` quote is free to break out of the string:

```
input:   %bf%27 OR 1=1-- 
escaped: %bf%5c%27 OR 1=1--     (0xBF5C = one GBK character, quote survives)
```

The preconditions are specific: the connection charset is GBK (or another vulnerable multibyte encoding), and the escaping is charset-unaware, such as `addslashes()` or `mysql_real_escape_string()` called before the charset is set correctly. Properly setting the charset with `mysqli_set_charset('gbk')` makes `mysql_real_escape_string` charset-aware and closes the gap, so this mainly affects older or misconfigured stacks. Other lead bytes such as `%bf`, `%df`, and `%a1` work on the same principle.

## Tools

- **sqlmap**: `unmagicquotes` tamper script for wide-byte escaping bypass.
- **Burp Repeater**: craft the `%bf%27` multibyte payload manually.

## References

- MySQL Reference Manual: character sets, `mysql_real_escape_string` charset handling
- OWASP Testing Guide: Testing for SQL Injection
