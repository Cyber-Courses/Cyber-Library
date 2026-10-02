---
title: "PostgreSQL command execution via untrusted-language functions"
description: "Defining a function in plpythonu, plperlu, or C from a superuser PostgreSQL injection to run OS commands when COPY FROM PROGRAM is not available."
keywords:
  - plpythonu
  - plperlu
  - CREATE FUNCTION
  - libc system
  - PostgreSQL RCE
---

# Untrusted language function

When `COPY ... FROM PROGRAM` is blocked but the role is a superuser, defining a function in an untrusted language gives command execution. Untrusted languages (`plpythonu`, `plperlu`, and C) run with no sandbox, so their functions can call out to the operating system. Creating them requires a superuser.

With an untrusted procedural language installed, wrap a subprocess call. The modern extension is PL/Python 3, registered as `plpython3u` (the old Python 2 `plpythonu` is gone on maintained releases), so confirm the available language in `pg_language` first:

```sql
' UNION SELECT string_agg(lanname,','),NULL,NULL FROM pg_language-- 
'; CREATE OR REPLACE FUNCTION exec(cmd text) RETURNS text AS $$ import subprocess; return subprocess.check_output(cmd, shell=True).decode() $$ LANGUAGE plpython3u-- 
' UNION SELECT exec('id'),NULL,NULL-- 
```

`plperlu` is the equivalent where Perl is the installed untrusted language.

A C-language function is the fallback when no untrusted procedural language is present, but it is not as simple as binding to libc: PostgreSQL rejects any shared object that lacks its module magic block (`incompatible library ... missing magic block`), so `CREATE FUNCTION ... AS '/lib/.../libc.so.6','system' LANGUAGE c` fails on current versions. The real C route is to compile a small PostgreSQL extension (a `.so` built with `PG_MODULE_MAGIC` exporting a command-exec function) and load it with `CREATE FUNCTION`, which is why command execution in practice leans on `COPY ... FROM PROGRAM` or an untrusted procedural language rather than a bare libc binding.

Every variant here runs as the PostgreSQL process account and needs a superuser, so enumerate the role first; against a non-superuser, `COPY ... FROM PROGRAM` through the `pg_execute_server_program` role is the only command route.

## Tools

- **sqlmap**: automates command execution against privileged PostgreSQL roles.
- **psql**: official client to create and call the untrusted-language function.

## References

- PostgreSQL Documentation: CREATE FUNCTION, procedural languages, C-language functions
- OWASP Testing Guide: Testing for SQL Injection
