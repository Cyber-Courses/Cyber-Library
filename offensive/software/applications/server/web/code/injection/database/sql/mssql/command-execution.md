---
title: "MSSQL command execution via SQL: xp_cmdshell, OLE automation, and external scripts"
description: Reaching host command execution from SQL Server through xp_cmdshell, sp_OACreate, sp_execute_external_script, and chained file or job primitives during SQL injection.
keywords:
  - xp_cmdshell
  - sp_OACreate
  - sp_execute_external_script
  - SQL Server command execution
---
# Command execution (MSSQL)

**Command execution** on SQL Server usually means **`xp_cmdshell`**, **`sp_OACreate`** (COM), or **external script** endpoints on supported editions. What you get depends on **feature state** (`sp_configure`), **patch level**, and the **privilege** of the session your injection runs in, web apps are often low-priv, but misconfig and chained privesc are common stories.

## Context

Host command execution on SQL Server usually means **`xp_cmdshell`** (if on), COM via **`sp_OACreate`**, or external script endpoints, requires high privilege and enabled features.
## Technique

Workflow: confirm **sysadmin** or equivalent → check `xp_cmdshell` config → `EXEC xp_cmdshell 'cmd /c ...'`. If disabled, hunt misconfig, agent jobs, or linked-server hop.
## Practice

- Read `sys.configurations` for config_id 0 (xp_cmdshell).
- If shell fails, chain **file write** + startup folder or scheduled task per target OS policy.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
