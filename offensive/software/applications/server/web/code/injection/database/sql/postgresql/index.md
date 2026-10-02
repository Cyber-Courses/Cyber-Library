---
title: "PostgreSQL SQL injection: techniques, payloads, and data extraction"
description: "Exploiting SQL injection against PostgreSQL: dollar-quoting, string casts for error leaks, pg_sleep timing, stacked queries, pg_read_file, and COPY ... PROGRAM command execution."
keywords:
  - PostgreSQL SQL injection
  - pg_sleep
  - COPY PROGRAM
  - dollar quoting
  - pg_catalog
  - error based cast
---

# PostgreSQL

PostgreSQL has a rich dialect that changes several injection techniques compared with MySQL, and it tends to reward a successful injection with more power because its file and command primitives are built in.

Comments are `--` and `/* */`. Strings concatenate with the standard `||` operator, and dollar-quoting (`$$text$$` or `$tag$text$tag$`) provides a quote-free string literal that is useful for evading filters. Type handling is strict: `UNION` requires the column types on both sides to match or be explicitly cast, so `NULL::text` and `value::text` casts appear throughout union payloads.

Unlike the common MySQL drivers, PostgreSQL commonly allows stacked queries: a `;`-separated second statement often executes, which opens `CREATE`, `COPY`, and DDL from a single injection. The catalog lives in both the SQL-standard `information_schema` and the native `pg_catalog` (`pg_tables`, `pg_class`, `pg_namespace`, `pg_roles`, `pg_database`), and `pg_catalog` is often reachable when `information_schema` is filtered.

Power depends on the role. A superuser (or a role granted `pg_read_server_files`/`pg_execute_server_program` in version 11 and later) can read files with `pg_read_file()`, write them with `COPY ... TO`, and run OS commands with `COPY ... FROM PROGRAM`. Checking `current_setting('is_superuser')` early decides which of those routes are open.

## Techniques

- **[Enumeration](enumeration.md)**: fingerprint the version, current context, and role.
- **[Authentication bypass](authentication-bypass.md)**: subvert a login built from the credential fields.
- **[Union-based](union-based/index.md)**: append a `UNION SELECT` with matching casts.
- **[Error-based](error-based/index.md)**: leak values through type-cast errors.
- **[Boolean blind](blind/index.md)**: infer data from true/false response differences.
- **[Time-based](time-based.md)**: infer data from `pg_sleep` delays.
- **[Stacked queries](stacked-query.md)**: run extra statements after a `;`.
- **[Privileges](privileges.md)**: read the role and its grants.
- **[File manipulation](file-manipulation.md)**: read and write files with `pg_read_file` and `COPY`.
- **[Out-of-band](out-of-band.md)**: exfiltrate over DNS via `COPY ... PROGRAM`.
- **[Command execution](command-execution/index.md)**: run OS commands through `COPY PROGRAM` or an untrusted-language function.
- **[WAF bypass](waf-bypass.md)**: `CHR()`, dollar-quoting, and catalog alternatives past filters.

## References

- PostgreSQL Documentation: system catalogs, functions, and COPY
- OWASP Testing Guide: Testing for SQL Injection
