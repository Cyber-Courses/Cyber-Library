---
title: "Remote code execution through SQLite injection"
description: "Reaching code execution from SQLite injection via load_extension loading a shared library, and the ATTACH web-shell written to a served directory."
keywords:
  - load_extension
  - SQLite RCE
  - shared library
  - ATTACH webshell
  - enable_load_extension
---

# Remote code execution

SQLite has two routes to code execution from injection, both with clear preconditions.

`load_extension(path)` loads a shared library and calls its entry point, which runs native code. This is the direct route, but it is off by default at two levels: many builds compile with loadable extensions disabled entirely, and even when compiled in, extension loading must be enabled at runtime through the C API (`sqlite3_enable_load_extension`) or the `enable_load_extension` PRAGMA before the SQL `load_extension()` function will work. Where an application has enabled it, a malicious library reached through a writable or attacker-supplied path gives execution:

```sql
' ; SELECT load_extension('/tmp/evil')-- 
```

The entry point (named for the file, for example `sqlite3_evil_init`) runs when the library loads, so merely loading it executes the attacker's code as the application process.

The second route does not need `load_extension` at all: it chains the file-write primitive with the web layer. Writing a PHP (or other server-side) payload into the document root with `ATTACH DATABASE`, as shown on the file-manipulation page, produces a script the web server executes on request, giving command execution through HTTP rather than through SQLite itself. This is often the more practical path, since `load_extension` is usually disabled, while a writable web root plus stacked execution is a common configuration in the small apps that embed SQLite.

## References

- SQLite Documentation: load_extension, enable_load_extension, run-time loadable extensions
- OWASP Testing Guide: Testing for SQL Injection
