---
title: "MSSQL database credential exposure via SQL injection: syslogins and hash metadata"
description: How SQL Server login metadata can be queried through injection, and why application accounts should not be able to read password hashes or legacy catalog views.
keywords:
  - sys.syslogins
  - LOGINPROPERTY
  - password hash
  - SQL Server credentials
---
# Database credentials (MSSQL)

Attackers with **SELECT** access to catalog views may enumerate **logins**, **roles**, and sometimes **hash material** suitable for offline cracking, depending on version and permissions. Legacy and modern catalog names differ (`syslogins`, `sys.sql_logins`, `sys.server_principals`).

## Context

MSSQL stores login metadata in `sys.server_principals` / legacy catalogs; password hashes may be reachable for offline cracking depending on version and rights.
## Technique

Select from **`sys.sql_logins`** with `LOGINPROPERTY` for hash material when `CONTROL SERVER` or similar is not required, often you need elevated rights.
## Practice

- Map effective privileges before burning time on hash pulls.
- Export hashes for **hashcat** / **John** rules tuned to SQL Server formats.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
