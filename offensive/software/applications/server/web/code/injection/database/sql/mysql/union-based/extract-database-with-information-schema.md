---
title: "Enumerating databases and tables via information_schema in MySQL UNION SQL injection"
description: Using information_schema.tables and information_schema.columns in UNION-based MySQL injection to list schemas and columns.
keywords:
  - information_schema
  - MySQL SQL injection
  - table extraction
---

# information_schema extraction

## Context

When the MySQL user has read access to **`information_schema`**, a `UNION SELECT` can pull **`table_schema`**, **`table_name`**, and **`column_name`** from **`information_schema.tables`** and **`information_schema.columns`**. This accelerates mapping the database for follow-on reads. Use only where you have authorization.

## Theory

Typical extraction shape: union the vulnerable query with a select that returns string columns from `information_schema.tables` where `table_schema` is not in `('information_schema','mysql','performance_schema','sys')` on MySQL 8 defaults. Concatenate multiple fields with `CONCAT_WS` or `GROUP_CONCAT` if the injection point returns a single column to the page.

## Practice

### List non-system schemas

- After column count is known, place in one displayable column an expression such as:

```sql
SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata WHERE schema_name NOT IN ('information_schema','mysql','performance_schema','sys')
```

Adapt wrapping parentheses and comment style to the injection context (`'`, `"`, or numeric).

### Map one application schema

- Filter `table_schema='targetdb'` in `information_schema.tables` and pull `table_name`. Then join or query `information_schema.columns` for `column_name` and `data_type`.

## Tools

- **sqlmap**
- **Burp Suite**
