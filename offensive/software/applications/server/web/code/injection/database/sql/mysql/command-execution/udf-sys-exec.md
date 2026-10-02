---
title: "MySQL command execution via the lib_mysqludf_sys UDF"
description: "Loading the lib_mysqludf_sys library and registering sys_exec/sys_eval to run OS commands directly from a privileged MySQL injection."
keywords:
  - lib_mysqludf_sys
  - sys_exec
  - sys_eval
  - user defined function
  - MySQL UDF RCE
---

# UDF sys_exec

A user-defined function extends MySQL with native code loaded from a shared library in the plugin directory. The `lib_mysqludf_sys` library exposes `sys_exec` (run a command) and `sys_eval` (run a command and return its output), which give command execution as the MySQL service account straight from SQL.

The chain has three steps, each needing privilege:

1. Write the library into the plugin directory with `INTO DUMPFILE` (needs `FILE`, a permissive `secure_file_priv`, and a writable `@@plugin_dir`).
2. Register the function, which needs `CREATE FUNCTION` (part of the admin privileges):

```sql
CREATE FUNCTION sys_exec RETURNS INT SONAME 'lib_mysqludf_sys.so';
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'lib_mysqludf_sys.so';
```

3. Call it:

```sql
SELECT sys_eval('id');
```

`sys_eval('id')` returns the command output as a string, so it reads back in-band; `sys_exec` returns only an exit status and is used for fire-and-forget actions such as adding a user or starting a reverse shell.

This route depends on stacked queries or a query context that allows `CREATE FUNCTION`, which the common PHP drivers do not provide, so in practice it is reached through an admin console, a multi-statement client, or an injection in a context that permits multiple statements. The payoff is execution as the database service account rather than the lower-privileged web user a web shell runs as.

## Tools

- sqlmap (`--os-shell` automates the OUTFILE and UDF routes)
- lib_mysqludf_sys

## References

- MySQL Reference Manual: CREATE FUNCTION (UDF), `plugin_dir`
- OWASP Testing Guide: Testing for SQL Injection
