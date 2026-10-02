---
title: "Error-based SQL injection in MySQL"
description: "Forcing MySQL to leak query results inside error messages using XPath functions EXTRACTVALUE and UPDATEXML and the FLOOR/RAND double-query technique."
keywords:
  - error based SQL injection
  - EXTRACTVALUE
  - UPDATEXML
  - double query injection
  - MySQL error leak
---

# Error-based

Error-based injection is useful when the application does not print query rows but does reflect database error text. The trick is to make a function fail in a way that embeds a subquery's result inside the error string, then read the value from the displayed error.

MySQL has two reliable error channels. The XPath functions `EXTRACTVALUE()` and `UPDATEXML()` (added in MySQL 5.1.5 and still present in MySQL 8.0, as well as in MariaDB) reject malformed XPath and quote the offending text back in the message, so wrapping a subquery in an invalid XPath expression leaks its output. The older `FLOOR(RAND())` with `GROUP BY` approach forces a duplicate-key error that also contains the value, and works on versions where the XPath functions are unavailable.

Both XPath functions truncate the reflected text to about 32 characters. Longer values are read in windows with `SUBSTRING(payload, 1, 32)`, then `SUBSTRING(payload, 32, 32)`, and so on, reassembling the pieces.

## Pages

- **[EXTRACTVALUE](extractvalue.md)**: leak via an invalid XPath in a two-argument function.
- **[UPDATEXML](updatexml.md)**: the same channel through the three-argument function.

## References

- MySQL Reference Manual: EXTRACTVALUE, UPDATEXML, XPath functions
- OWASP Testing Guide: Testing for SQL Injection
