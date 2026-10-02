---
title: "Detecting column count for a MySQL UNION injection"
description: "Finding the number of columns a MySQL query projects, using ORDER BY ordinal probing and incremental UNION SELECT NULL, before extracting data."
keywords:
  - column count detection
  - ORDER BY injection
  - UNION SELECT NULL
  - MySQL union columns
---

# Column count detection

A `UNION SELECT` only runs if it projects exactly as many columns as the original query. The first step of any union attack is therefore to count those columns. Two probes do this.

The `ORDER BY` method asks the database to sort by an ordinal column number and watches for the point where the number exceeds the real column count:

```sql
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3-- 
```

Sorting succeeds while the ordinal is valid and fails with `Unknown column '4' in 'order clause'` once it passes the last column. The highest number that still works is the column count. This is quiet (no extra rows) and works even when the page shows only a status, not data.

The `UNION SELECT NULL` method appends increasing lists of `NULL` until the column counts match and the query runs without a type or count error:

```sql
' UNION SELECT NULL-- 
' UNION SELECT NULL,NULL-- 
' UNION SELECT NULL,NULL,NULL-- 
```

`NULL` is used because it is compatible with any column type, so only the count, not the types, decides success. The list length that stops producing `The used SELECT statements have a different number of columns` is the answer, and the same request already tells you the union is live.

With the count known, replace the `NULL`s one at a time with a marker such as a number or a string to learn which positions are echoed back on the page. Those reflected positions are where extracted data must go.

## Tools

- **sqlmap**: automates column-count detection and union alignment.
- **Burp Repeater**: run `ORDER BY` and `UNION SELECT NULL` probes by hand.

## References

- MySQL Reference Manual: UNION clause, ORDER BY
- PortSwigger Web Security Academy: SQL injection UNION attacks
