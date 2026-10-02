---
title: "MSSQL file read and write via SQL: BULK, OPENROWSET, and bulk operations"
description: File-oriented features in SQL Server that attackers may reach through injection—bulk insert, ad hoc distributed queries, and operational prerequisites.
keywords:
  - BULK INSERT
  - OPENROWSET
  - SQL Server file read
---
# File manipulation (MSSQL)

SQL Server can **read** and **write** files through **bulk operations**, `OPENROWSET`, and sometimes chained **command execution**. Effectiveness depends on **privileges** (`ADMINISTER BULK OPERATIONS`, `CONTROL SERVER`), **trustworthy** database settings, and filesystem layout.

## Context

Read/write paths include **`OPENROWSET` BULK**, **`BULK INSERT`**, OLE automation, and cmd-assisted writes when shell exists.
## Technique

Pick the primitive that matches grants: bulk roles for file read, `xp_cmdshell`/`certutil` chains for exfil when outbound HTTP is blocked.
## Practice

- Test `OPENROWSET(BULK...)` for UNC or local paths the service account can read.
- Write webshells only where the web root is co-located and writable—validate layout first.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Command execution](command-execution.md)
