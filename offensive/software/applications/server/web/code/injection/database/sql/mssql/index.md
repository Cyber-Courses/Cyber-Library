---
title: "Microsoft SQL Server (MSSQL) SQL injection: techniques and payloads"
description: "Exploiting SQL injection against Microsoft SQL Server: stacked queries, conversion-error leaks, WAITFOR timing, xp_cmdshell, linked servers, and the sys catalog."
keywords:
  - MSSQL SQL injection
  - T-SQL injection
  - xp_cmdshell
  - WAITFOR DELAY
  - conversion error
  - linked servers
---

# MSSQL

Microsoft SQL Server runs the T-SQL dialect, and a successful injection against it is often high-impact because command execution, file access, and lateral movement to other servers are all reachable from SQL.

Comments are `--` and `/* */`. Strings concatenate with `+` (and `CONCAT()` from 2012), and `CHAR()`/`NCHAR()` build strings without quotes. MSSQL commonly permits stacked queries: a `;`-separated second statement usually runs, which makes `EXEC`, DDL, and configuration changes reachable from one injection. The catalog lives in the `sys` schema (`sys.databases`, `sys.tables`, `sys.columns`, `sys.sql_logins`) and in `information_schema`, and server metadata comes from `@@version`, `SERVERPROPERTY()`, `DB_NAME()`, and `SYSTEM_USER`.

Two dialect details shape payloads. There is no `LIMIT`; row limiting is `TOP n` or `OFFSET ... FETCH` (2012+), so blind and union payloads use `TOP 1`. And aggregation into one cell uses `STRING_AGG()` (2017+) or the `FOR XML PATH('')` trick on older versions, in place of MySQL's `GROUP_CONCAT`.

Impact depends on the login's server role. A member of `sysadmin` can enable and run `xp_cmdshell`, read and write files, and pivot through linked servers, so `IS_SRVROLEMEMBER('sysadmin')` is checked early.

## Techniques

- **[Enumeration](enumeration.md)**: version, current context, and server role.
- **[Authentication bypass](authentication-bypass.md)**: subvert a login built from the credential fields.
- **[Union-based](union-based.md)**: append a `UNION SELECT` with matching types.
- **[Error-based](error-based.md)**: leak values through conversion errors.
- **[Blind](blind.md)**: infer data from boolean response differences.
- **[Time-based](time-based.md)**: infer data with `WAITFOR DELAY`.
- **[Stacked queries](stacked-query.md)**: run extra statements after a `;`.
- **[Privileges](privileges.md)**: read the login's server and database roles.
- **[Database credentials](database-credentials.md)**: dump login password hashes.
- **[File manipulation](file-manipulation.md)**: read and write files with OPENROWSET and bcp.
- **[Out-of-band](out-of-band.md)**: exfiltrate and capture hashes over UNC with `xp_dirtree`.
- **[Command execution](command-execution.md)**: run OS commands with `xp_cmdshell` or OLE automation.
- **[Trusted links](trusted-links.md)**: pivot to linked servers with `openquery` and RPC.

## Tools

- **sqlmap**: automated detection and exploitation of SQL Server injection across all techniques.
- **ghauri**: fast alternative with strong WAF evasion.
- **sqlcmd**: official client for validating T-SQL payloads directly.

## References

- Microsoft SQL Server Documentation: system catalog views, functions, configuration
- OWASP Testing Guide: Testing for SQL Injection
