---
title: "MSSQL list permissions: HAS_PERMS_BY_NAME and fn_my_permissions"
description: Enumerating effective SQL Server permissions through catalog functions—useful for audits and for understanding injection reach.
keywords:
  - HAS_PERMS_BY_NAME
  - fn_my_permissions
  - IS_SRVROLEMEMBER
---
# List permissions (MSSQL)

**Security catalog functions** list what the **current principal** can do on objects and at **server** scope—use them to prioritize **xp_cmdshell**, **bulk**, **linked server**, and **IMPERSONATE** paths before burning time on dead ends.

## Context

Enumerate object and server permissions to see which primitives (bulk, Ole Automation, linked servers) are in play.
## Technique

Walk `fn_my_permissions` for `%` object class or spot-check sensitive objects.
## Practice

- Script outputs for reporting—customers want a clear blast-radius matrix.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Privileges (MSSQL)](index.md)
