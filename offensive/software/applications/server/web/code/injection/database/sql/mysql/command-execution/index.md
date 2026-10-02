---
title: "From MySQL injection to command execution"
description: "The two routes from a privileged MySQL injection to code execution: writing a web shell with INTO OUTFILE/DUMPFILE, or loading a sys_exec UDF."
keywords:
  - MySQL command execution
  - INTO OUTFILE webshell
  - INTO DUMPFILE
  - lib_mysqludf_sys
  - sys_exec
---

# Command execution

MySQL has no built-in function that runs an OS command, so code execution is reached indirectly from a privileged injection. There are two routes, and both depend on `FILE` privilege and a `secure_file_priv` that permits writes.

The first route writes a web shell into a directory the web server serves. `SELECT ... INTO OUTFILE` and `INTO DUMPFILE` write query output to a file on the host; aiming that at the web root drops a script that is then run over HTTP, which converts database access into application-level command execution.

The second route extends the server itself with a user-defined function. Writing the `lib_mysqludf_sys` shared library into the plugin directory and registering `sys_exec`/`sys_eval` gives a function that runs shell commands directly from SQL. It needs more privilege (write access to the plugin directory and `CREATE FUNCTION`) but yields command execution as the MySQL service account without touching the web layer.

## Pages

- **[INTO OUTFILE](into-outfile.md)**: write a web shell into the document root.
- **[INTO DUMPFILE](into-dumpfile.md)**: write exact bytes, used for binaries such as a UDF library.
- **[UDF sys_exec](udf-sys-exec.md)**: load `lib_mysqludf_sys` for direct command execution.

## References

- MySQL Reference Manual: SELECT INTO, user-defined functions, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
