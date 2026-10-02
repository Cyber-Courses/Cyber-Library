---
title: "Reading server files through MySQL injection with LOAD_FILE"
description: "Using LOAD_FILE to read files from the database host through a MySQL injection, and the FILE privilege and secure_file_priv conditions that gate it."
keywords:
  - LOAD_FILE
  - file read
  - secure_file_priv
  - FILE privilege
  - MySQL file disclosure
---

# File read

`LOAD_FILE(path)` returns the contents of a file on the database server as a string, which makes it a direct read primitive over anything the MySQL account can open. It is commonly used to pull configuration files, source code, and credentials from the host.

Place it in a reflected column:

```sql
' UNION SELECT NULL,LOAD_FILE('/etc/passwd'),NULL-- 
```

Three conditions must all hold, or `LOAD_FILE` silently returns `NULL`:

- The account has the `FILE` privilege.
- `secure_file_priv` permits the path. Check it with `SELECT @@secure_file_priv`: an empty value means no restriction, a directory means reads are confined to that directory, and `NULL` means file operations are disabled entirely.
- The OS file is readable by the MySQL process user and within `max_allowed_packet` in size.

Because a blocked read and a missing file both yield `NULL`, confirm the privilege first by reading a file that is known to exist and be world-readable. Binary or non-UTF-8 files come back garbled through a text column, so wrap them in `HEX(LOAD_FILE(...))` and decode client-side.

## References

- MySQL Reference Manual: LOAD_FILE, `secure_file_priv`, FILE privilege
- OWASP Testing Guide: Testing for SQL Injection
