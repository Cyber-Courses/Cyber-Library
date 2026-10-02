---
title: "WAF and filter bypass for PostgreSQL injection"
description: "Evading filters in PostgreSQL injection with CHR() string building, dollar-quoting, concatenation, and pg_catalog alternatives to blocked names."
keywords:
  - WAF bypass
  - CHR function
  - dollar quoting
  - PostgreSQL filter evasion
  - pg_catalog alternative
---

# WAF bypass

PostgreSQL's dialect gives several ways around signature filters that block quotes, keywords, or the string `information_schema`.

Quotes are avoided by building strings from character codes with `CHR()` joined by `||`, or with dollar-quoting, which needs no quote character at all:

```sql
-- '/etc/passwd' without quotes
CHR(47)||CHR(101)||CHR(116)||CHR(99)||CHR(47)||CHR(112)||CHR(97)||CHR(115)||CHR(115)||CHR(119)||CHR(100)
-- dollar-quoted literal
$$/etc/passwd$$
```

Keyword signatures are split with concatenation and inline comments, since PostgreSQL ignores `/**/` between tokens:

```sql
' UNION/**/SELECT/**/NULL,version(),NULL-- 
```

When `information_schema` is blocklisted, the native `pg_catalog` reaches the same data under different names: `pg_tables` for tables, `pg_namespace` for schemas, and `pg_class` joined to `pg_attribute` for columns. When `version()` is filtered, `current_setting('server_version')` returns the version string, and `current_setting('server_version_num')` returns it as a number.

Numeric and encoding tricks also help: `CHR()` and `convert_from(decode('...','hex'),'UTF8')` reconstruct filtered strings, and casting with `::text` avoids explicit `CAST` keywords. As with other engines, the approach is to reach the same object or value through a synonym the filter does not know, rather than to defeat the filter head-on.

## References

- PostgreSQL Documentation: `chr`, dollar-quoted strings, `current_setting`, system catalogs
- OWASP Testing Guide: Testing for SQL Injection
