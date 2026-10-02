---
title: "Writing exact bytes with MySQL INTO DUMPFILE"
description: "Using SELECT INTO DUMPFILE to write a single row verbatim from a MySQL injection, the binary-safe write used to drop a UDF shared library."
keywords:
  - INTO DUMPFILE
  - binary file write
  - UDF library
  - MySQL file write
  - plugin directory
---

# INTO DUMPFILE

`SELECT ... INTO DUMPFILE '/path/file'` writes a single row to a file with no row or column formatting, so the bytes land exactly as selected. This is the binary-safe counterpart to `INTO OUTFILE`: `OUTFILE` adds terminators and escaping that would corrupt a binary, while `DUMPFILE` does not. Its main use is dropping a compiled artifact, most often the shared library for a command-execution UDF.

The content is supplied as a hex literal so arbitrary bytes survive transport, and the target is the server's plugin directory:

```sql
' UNION SELECT 0x7f454c46...  INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so'-- 
```

Find the plugin directory first so the library lands where `CREATE FUNCTION` will look for it:

```sql
' UNION SELECT @@plugin_dir,NULL,NULL-- 
```

`DUMPFILE` writes only one row, so it cannot dump a multi-row result, and like `OUTFILE` it requires `FILE` privilege, a permissive `secure_file_priv`, a writable destination, and a non-existent target file. With the library in place, the UDF is registered and called as shown in the `sys_exec` page.

## References

- MySQL Reference Manual: SELECT INTO DUMPFILE, `plugin_dir`, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
