---
title: "Error-based SQL injection in SQLite"
description: "Leaking SQLite data through primitives that genuinely raise errors, since divide-by-zero returns NULL and casts coerce silently rather than erroring."
keywords:
  - error based SQL injection
  - SQLite errors
  - undefined function
  - integer overflow
  - CAST coercion
---

# Error-based

Error-based injection in SQLite needs care, because the tricks that work elsewhere do not raise errors here. Dividing by zero returns `NULL` (not an error), and `CAST('abc' AS INTEGER)` silently yields `0`, so neither leaks data. The channel instead relies on primitives that genuinely fail, embedding the target value in the error text.

Several operations raise a usable error:

```sql
-- undefined function: the message echoes the name
' AND 1=(SELECT upper(no_such_fn()))-- 
-- integer overflow
' AND 1=abs(-9223372036854775808)-- 
-- malformed JSON (JSON1 extension)
' AND 1=(SELECT json('x'))-- 
```

These confirm injection and fingerprint the engine, but SQLite does not offer a clean, general way to splice an arbitrary query result into an error message the way PostgreSQL's cast error or SQL Server's conversion error do. Its error surface is narrow and the messages do not reliably include a full attacker-chosen value. In practice, therefore, error output is used here to prove the point is live (an `unrecognized token`, `no such column`, or `no such function` error), and the actual data is extracted with boolean or time-based inference, which are the dependable channels against SQLite.

## References

- SQLite Documentation: core functions, JSON1, error messages
- OWASP Testing Guide: Testing for SQL Injection
