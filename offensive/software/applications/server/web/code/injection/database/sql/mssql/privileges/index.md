---
title: "MSSQL privileges in SQL injection: effective permissions and role abuse"
description: Listing and escalating privileges on SQL Server—relevant when injection can query security catalog functions or alter server roles.
keywords:
  - HAS_PERMS_BY_NAME
  - fn_my_permissions
  - sysadmin
---
# Privileges (MSSQL)

Map **effective permissions** to see whether you can reach **`sysadmin`**, **OLE automation**, **linked servers**, or **bulk** file ops. SQL Server exposes **`HAS_PERMS_BY_NAME`**, **`fn_my_permissions`**, and **`IS_SRVROLEMEMBER`** from the injection context.

| Topic | Path |
|-------|------|
| List permissions | [List permissions](list-permissions.md) |
| Make user DBA | [Make user DBA](make-user-dba.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](../index.md)
