---
title: "Error-based SQL injection in MSSQL"
description: "Leaking SQL Server query results through type-conversion errors, which quote the offending value in the message with no length limit."
keywords:
  - error based SQL injection
  - conversion failed
  - CONVERT CAST
  - MSSQL error leak
---

# Error-based

When the application hides rows but reflects database errors, SQL Server leaks data through a type-conversion failure. Converting a string to `int` raises `Conversion failed when converting the varchar value '<value>' to data type int`, and the message quotes the value that failed, so a subquery placed in the conversion prints its result.

Place the subquery in a `CONVERT` (or `CAST`) to `int`:

```sql
' AND 1=CONVERT(int,(SELECT @@version))-- 
```

The response carries `Conversion failed when converting the varchar value 'Microsoft SQL Server 2019 ...' to data type int`. Any single-value subquery works:

```sql
' AND 1=CONVERT(int,(SELECT TOP 1 name FROM sys.tables))-- 
' AND 1=CONVERT(int,(SELECT TOP 1 name+':'+master.dbo.fn_varbintohexstr(password_hash) FROM sys.sql_logins))-- 
```

Note that a password hash is `varbinary`: render it with `fn_varbintohexstr` (or `CONVERT(varchar(max), hash, 2)`) rather than a plain `CAST` to `varchar`, which would reinterpret the raw bytes as text and produce unusable, NUL-containing output.

Unlike MySQL's XPath channel, SQL Server does not cap the value at ~32 characters, so a row usually returns in one error. It is not unlimited, though: error messages have a finite length and a very long value or aggregate can be truncated, so for large dumps read in windows with `SUBSTRING((SELECT ...),1,2000)` and page through. Keep the inner query to a single value with `TOP 1` or an aggregate, since a multi-row subquery used as a scalar raises a different error that carries no data.

## Tools

- **sqlmap**: automated error-based extraction through conversion failures.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- Microsoft SQL Server Documentation: CONVERT and CAST, data type conversion
- OWASP Testing Guide: Testing for SQL Injection
