---
title: "Fingerprinting and enumeration in PostgreSQL injection"
description: "Orienting a PostgreSQL injection: confirming the engine, reading version and current context, and checking whether the role is a superuser."
keywords:
  - PostgreSQL fingerprinting
  - version function
  - current_user current_database
  - is_superuser
  - role enumeration
---

# Enumeration

Confirm the engine is PostgreSQL and read the role context before choosing a technique, because the file, out-of-band, and command routes all depend on whether the role is privileged.

`version()` fingerprints PostgreSQL unambiguously (its output begins with the literal `PostgreSQL`, which also separates it from MySQL whose version starts with a digit), and the identity functions give the current role and database:

```sql
' UNION SELECT version(),current_user,current_database()-- 
```

`current_user` is the role used for permission checks, `session_user` is the role that connected, and `current_database()` names the database. The numeric version for comparisons is `current_setting('server_version_num')` (for example `160001`), which avoids parsing the text banner.

The single most important fact is whether the role is a superuser, since that decides file read, file write, and command execution:

```sql
' UNION SELECT current_setting('is_superuser'),NULL,NULL-- 
```

A value of `on` means the full file and program primitives are available. On version 11 and later a non-superuser can still reach some of them if it belongs to the `pg_read_server_files`, `pg_write_server_files`, or `pg_execute_server_program` roles, which the privileges page enumerates. With the version and role known, the remaining techniques apply with the right expectations.

## Tools

- **sqlmap**: fingerprints the engine and reads version, role, and superuser status.
- **psql**: official client to confirm `version()`, `current_user`, and `is_superuser` directly.

## References

- PostgreSQL Documentation: system information functions, `current_setting`
- OWASP Testing Guide: Testing for SQL Injection
