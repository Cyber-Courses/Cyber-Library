---
title: "Union-based SQL injection in PostgreSQL"
order: 10
description: "Appending UNION SELECT in PostgreSQL to extract data, handling its strict type matching with explicit casts and reading the pg_catalog and information_schema catalogs."
keywords:
  - union based SQL injection
  - UNION SELECT PostgreSQL
  - type cast
  - pg_catalog
  - string_agg
---

# Union-based

A `UNION SELECT` appends attacker-chosen rows to a returned result. PostgreSQL adds one wrinkle over MySQL: its type system is strict, so each column in the appended `SELECT` must match the original column's type or be cast explicitly. Casting everything to `text` with `::text` (or selecting `NULL`, which fits any type) sidesteps type mismatches.

The flow is the same as elsewhere: detect the column count, find a reflected text column, then read the catalog and data through it. PostgreSQL exposes schema information in both the SQL-standard `information_schema` and the native `pg_catalog`, and `string_agg()` is its equivalent of `GROUP_CONCAT` for collapsing many rows into one cell.

```sql
' UNION SELECT NULL,string_agg(table_name,','),NULL FROM information_schema.tables WHERE table_schema='public'-- 
```

## Pages

- **[Detect column count](detect-column-count.md)**: `ORDER BY` and `UNION SELECT NULL` probing, with casts.
- **[Extract schema and data](extract-schema.md)**: enumerate and dump via `information_schema` and `pg_catalog`.

## Tools

- **sqlmap**: automated union-based detection and extraction.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- PostgreSQL Documentation: UNION, type casts, `string_agg`
- OWASP Testing Guide: Testing for SQL Injection
