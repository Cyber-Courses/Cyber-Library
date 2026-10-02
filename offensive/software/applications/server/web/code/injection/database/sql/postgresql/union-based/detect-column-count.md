---
title: "Detecting column count for a PostgreSQL UNION injection"
description: "Finding the number of columns a PostgreSQL query projects with ORDER BY ordinal probing and incremental UNION SELECT NULL, before extracting data."
keywords:
  - column count detection
  - ORDER BY injection
  - UNION SELECT NULL
  - PostgreSQL union columns
---

# Column count detection

A `UNION SELECT` runs only when it projects the same number of columns as the original query, so the count comes first. Two probes find it.

`ORDER BY` with an ordinal column number succeeds until it exceeds the real count:

```sql
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3-- 
```

The query works while the ordinal is valid and fails with `ORDER BY position N is not in select list` once it is too high. The last working number is the column count.

The `UNION SELECT NULL` probe appends increasing `NULL` lists until the counts match:

```sql
' UNION SELECT NULL-- 
' UNION SELECT NULL,NULL-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`NULL` is used because it is compatible with any column type, so only the count matters. The wrong count fails with `each UNION query must have the same number of columns`. Once the counts align, replace each `NULL` in turn with a cast value such as `'a'::text` to learn which columns are rendered on the page; those positions are where extracted data goes. Because PostgreSQL enforces types in a `UNION`, keep non-reflected columns as `NULL` and cast the reflected one to `text`.

## References

- PostgreSQL Documentation: UNION, ORDER BY, type casts
- PortSwigger Web Security Academy: SQL injection UNION attacks
