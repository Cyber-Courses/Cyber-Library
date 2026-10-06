---
title: "PostgreSQL"
order: 2
description: "The offensive surface of PostgreSQL reached as a service: authenticating, executing operating-system commands through COPY FROM PROGRAM and untrusted procedural languages, reading and writing host files, and escalating to superuser."
keywords:
  - PostgreSQL
  - COPY FROM PROGRAM
  - plpython
  - lo_export
  - superuser
---

# PostgreSQL

PostgreSQL (port 5432) is a frequent target because, as a **superuser**, it will run operating-system commands and read or write arbitrary files on the host, all through documented features. Even as a non-superuser there are file and function primitives worth reaching for, and several well-known paths promote an ordinary role to superuser.

## What to reach for

- **[Access](access.md)**: default and weak credentials, `trust` authentication, and reaching a login.
- **[Command execution](command-execution.md)**: `COPY ... FROM PROGRAM`, untrusted languages (`plpythonu`, `plperlu`), and C extensions.
- **[File access](file-access.md)**: `COPY`, large objects, and `pg_read_file` to read and write host files.

## References

- [HackTricks: pentesting PostgreSQL (5432)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
- [PostgreSQL: COPY](https://www.postgresql.org/docs/current/sql-copy.html)
