---
title: "Fingerprinting and enumeration in MSSQL injection"
description: "Orienting an MSSQL injection: confirming the engine, reading version and current context, and checking whether the login is a sysadmin."
keywords:
  - MSSQL fingerprinting
  - SERVERPROPERTY
  - SYSTEM_USER
  - IS_SRVROLEMEMBER
  - version detection
---

# Enumeration

Confirm the engine is SQL Server and read the login context before choosing a technique, because command execution, file access, and linked-server pivots all depend on the login's server role.

`@@version` and `SERVERPROPERTY()` fingerprint the server, and the identity functions give the current login and database:

```sql
' UNION SELECT @@version,SYSTEM_USER,DB_NAME()-- 
```

`SYSTEM_USER` is the login, `USER_NAME()` the database user it maps to, `DB_NAME()` the current database, and `@@SERVERNAME`/`HOST_NAME()` identify the instance and client host. For precise version logic, `SERVERPROPERTY('ProductVersion')`, `SERVERPROPERTY('ProductLevel')`, and `SERVERPROPERTY('Edition')` are cleaner than parsing the `@@version` banner.

The decisive check is server-role membership, since `sysadmin` unlocks `xp_cmdshell`, file access, and linked-server abuse:

```sql
' UNION SELECT IS_SRVROLEMEMBER('sysadmin'),IS_SRVROLEMEMBER('serveradmin'),CURRENT_USER-- 
```

`IS_SRVROLEMEMBER('sysadmin')` returns `1` when the login is a full administrator, which is the single most useful fact for planning the rest of the attack. A non-sysadmin can still be escalated in some configurations (a trustworthy database with an elevated owner, or impersonation via `EXECUTE AS`), which the privileges page covers. With the version and role known, the remaining techniques apply with the right expectations.

## References

- Microsoft SQL Server Documentation: SERVERPROPERTY, IS_SRVROLEMEMBER, system functions
- OWASP Testing Guide: Testing for SQL Injection
