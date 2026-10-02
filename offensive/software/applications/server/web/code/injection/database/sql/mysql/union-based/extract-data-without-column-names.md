---
title: "Extracting data without known column names using MySQL UNION and positional tricks"
description: Dumping cell values when table names are suspected but column names are unknown, positional subselects and concat tricks in MySQL.
keywords:
  - MySQL SQL injection
  - union injection
  - subquery
---

# Data without column names

## Context

Sometimes the **table** name leaks (`users`) but **column** names do not. Union and error channels may still move bytes out by selecting **positional** tuples or by **subqueries** that return a single scalar once you discover arity (e.g., `(SELECT * FROM users LIMIT 1)` in contexts that coerce row to scalar, MySQL behavior depends on **sql_mode** and query shape). Work in labs; validate legality for each target.

## Theory

Options include: discover names first (see [extract column names without information_schema](extract-column-names-without-information-schema.md)); use **`SELECT *`** inside a subquery in an error-prone cast to leak row **hex** or **substring** in the error text; or union into columns that display **hex(concat_ws(...))** of `(SELECT group_concat(...) FROM users)` after column count alignment using only positional knowledge from `LIMIT` and trial.

## Practice

### Single scalar via subquery

- When the injection sits in a numeric context, try:

```sql
(SELECT id FROM users LIMIT 1)
```

If `id` is wrong, iterate discovered columns from naming phase. If the server returns “Operand should contain N column(s)”, adjust to a one-column scalar subquery.

### GROUP_CONCAT row dump

- After columns are known, `UNION SELECT 1,group_concat(concat_ws(0x3a,login,pass)),3` style payloads (exact arity matches your column count) to place a string in a reflected column.

## Tools

- **sqlmap**
- **Burp Suite**
