---
title: "From PostgreSQL injection to command execution"
order: 2
description: "The two routes from a privileged PostgreSQL injection to OS command execution: COPY ... FROM PROGRAM, and functions in an untrusted procedural or C language."
keywords:
  - PostgreSQL command execution
  - COPY FROM PROGRAM
  - plpythonu
  - plperlu
  - RCE
---

# Command execution

PostgreSQL reaches OS command execution more directly than MySQL because the primitives are built into the server. Both routes need a privileged role, and both usually need a stacked query so a statement of their own can run.

The simplest route is `COPY ... FROM PROGRAM`, which runs a shell command and captures its output into a table. It requires a superuser or the `pg_execute_server_program` role and exists from PostgreSQL 9.3.

The second route defines a function in an untrusted language. Creating functions in `plpythonu`, `plperlu`, or C requires a superuser, and once created the function runs native code, so a wrapper around `subprocess` or `system()` gives command execution callable from SQL.

## Pages

- **[COPY FROM PROGRAM](copy-from-program.md)**: run a command and read its output through a table.
- **[Untrusted language function](language-function.md)**: define a `plpythonu`, `plperlu`, or C function that executes commands.

## Tools

- **sqlmap**: automates command execution with `--os-shell` over COPY FROM PROGRAM.
- **Metasploit Framework**: `postgres_payload` module for command execution on privileged roles.

## References

- PostgreSQL Documentation: COPY ... FROM PROGRAM, CREATE FUNCTION, procedural languages
- OWASP Testing Guide: Testing for SQL Injection
