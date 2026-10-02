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
' AND 1=CONVERT(int,(SELECT TOP 1 name+':'+CAST(password_hash AS varchar(max)) FROM sys.sql_logins))-- 
```

Unlike MySQL's XPath channel, SQL Server does not truncate the value, so a whole row or an aggregated dump comes back in one error. Keep the inner query to a single value with `TOP 1` or an aggregate, since a multi-row subquery used as a scalar raises a different error that carries no data. The same technique drives blind error-based extraction when only the presence or absence of the conversion error is observable.

## References

- Microsoft SQL Server Documentation: CONVERT and CAST, data type conversion
- OWASP Testing Guide: Testing for SQL Injection
