---
title: "Reading and writing files through Oracle injection"
description: "Oracle file access from injection: UTL_FILE with a directory object, DBMS_LOB for reads, and Java for arbitrary file operations, with the privileges each needs."
keywords:
  - UTL_FILE
  - directory object
  - DBMS_LOB
  - file read
  - file write Oracle
---

# File manipulation

A privileged Oracle session can read and write host files, which leads to configuration and credential disclosure and, through writing, to code execution. Each route needs an execute privilege and, for `UTL_FILE`, a directory object.

`UTL_FILE` reads and writes files inside a directory that a `DIRECTORY` object points to. With `CREATE ANY DIRECTORY` (or an existing readable directory), create or reuse one and read it. `CREATE DIRECTORY` is DDL, which cannot appear directly in a PL/SQL block, so it runs through `EXECUTE IMMEDIATE` inside the injected block (and Oracle does not stack plain statements, so this must be a PL/SQL context):

```sql
DECLARE f UTL_FILE.FILE_TYPE; s VARCHAR2(4000);
BEGIN
  EXECUTE IMMEDIATE 'CREATE OR REPLACE DIRECTORY d AS ''/etc''';
  f:=UTL_FILE.FOPEN('D','passwd','R'); UTL_FILE.GET_LINE(f,s); -- s now holds a line of /etc/passwd
END;
```

`UTL_FILE.FOPEN(..., 'W')` writes, which drops a script or a scheduled-task input into a writable directory. Reading through `UTL_FILE` is line-oriented; `DBMS_LOB` with a `BFILE` reads bytes for binary files.

The most general route is Java. A schema granted `java.io.FilePermission` can load a small Java class that reads or writes any path with the database server's OS privileges, bypassing the directory-object model entirely.

Because `UTL_FILE` and `CREATE DIRECTORY` are statements and PL/SQL, this route generally needs an injectable PL/SQL context or stacked execution (which Oracle grants only inside PL/SQL), plus the directory and execute privileges. Enumerate existing directories first with `SELECT directory_name, directory_path FROM all_directories` to find one already pointing somewhere useful.

## Tools

- **ODAT** (Oracle Database Attacking Tool): reads and writes server files via `UTL_FILE`, `DBMS_LOB`, and Java.
- **sqlplus** (or SQLcl): the Oracle client for creating directory objects and calling `UTL_FILE`.
- Manual testing with Burp Repeater and the Oracle client.

## References

- Oracle Database PL/SQL Packages and Types Reference: UTL_FILE, DBMS_LOB; CREATE DIRECTORY
- OWASP Testing Guide: Testing for SQL Injection
