---
title: "Error-based SQL injection in PostgreSQL"
description: "Forcing PostgreSQL to leak query results inside error messages through type-cast failures, which echo the offending value in full with no length limit."
keywords:
  - error based SQL injection
  - cast error
  - invalid input syntax
  - PostgreSQL error leak
---

# Error-based

Error-based injection suits applications that hide query rows but reflect database error text. PostgreSQL has no XPath error functions like MySQL's `EXTRACTVALUE`; instead the reliable channel is a type-cast failure. Casting a string to a numeric type raises `invalid input syntax for type integer: "<value>"`, and because the message quotes the value that failed to convert, a subquery placed in that cast leaks its result.

The advantage over MySQL's XPath channel is that PostgreSQL does not truncate the reflected value, so a whole row or an aggregated dump comes back in one error rather than in 32-character windows.

## Pages

- **[Cast error](cast-error.md)**: leak values through `CAST(... AS int)` conversion failures.

## References

- PostgreSQL Documentation: type casts, error messages
- OWASP Testing Guide: Testing for SQL Injection
