---
title: "Error-based MySQL injection with UPDATEXML"
description: "Leaking MySQL query results through XPath syntax errors raised by the three-argument UPDATEXML function, with the same 32-character truncation as EXTRACTVALUE."
keywords:
  - UPDATEXML
  - XPath syntax error
  - error based injection
  - MySQL data leak
---

# UPDATEXML

`UPDATEXML(xml_target, xpath_expression, new_value)` replaces part of an XML document selected by an XPath expression. Like `EXTRACTVALUE`, it rejects an invalid XPath argument with `XPATH syntax error: '<text>'` and echoes the text, so it is the second reliable error channel in MySQL. It is handy when `EXTRACTVALUE` is filtered but `UPDATEXML` is not.

Place the leaking expression in the second argument, prefixed with `~` so it is invalid XPath. The first and third arguments can be any valid values:

```sql
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT user())),1)-- 
```

The error returns `XPATH syntax error: '~root@localhost'`. Swap in any single-value subquery:

```sql
' AND UPDATEXML(1,CONCAT(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 1)),1)-- 
```

The same ~32-character truncation applies, so page through long values with `SUBSTRING` exactly as with `EXTRACTVALUE`, and keep the inner query to a single row.

## References

- MySQL Reference Manual: UPDATEXML, XPath functions
- OWASP Testing Guide: Testing for SQL Injection
