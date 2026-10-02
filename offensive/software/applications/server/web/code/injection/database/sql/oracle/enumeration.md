---
title: "Fingerprinting and enumeration in Oracle injection"
description: "Orienting an Oracle injection: confirming the engine, reading version and session context with SYS_CONTEXT, and listing the current user's privileges."
keywords:
  - Oracle fingerprinting
  - v$version
  - SYS_CONTEXT
  - session_privs
  - DUAL
---

# Enumeration

Confirm the engine is Oracle and read the session context before choosing a technique. The need for `FROM DUAL` is itself a fingerprint: a payload that only works with a `FROM` clause and fails without one points to Oracle.

Read the banner and identity. `v$version` returns several rows, so pin one with `ROWNUM`:

```sql
' UNION SELECT banner,NULL,NULL FROM v$version WHERE ROWNUM=1-- 
' UNION SELECT user,SYS_CONTEXT('USERENV','CURRENT_SCHEMA'),SYS_CONTEXT('USERENV','DB_NAME') FROM dual-- 
```

`SYS_CONTEXT('USERENV', ...)` is the richest source of session metadata: `SESSION_USER`, `CURRENT_USER`, `CURRENT_SCHEMA`, `DB_NAME`, `SERVER_HOST`, `OS_USER`, and `IP_ADDRESS`. `SELECT user FROM dual` gives the current schema owner, and `global_name` identifies the database.

Privileges decide which primitives are reachable, so list the session's roles and system privileges:

```sql
' UNION SELECT privilege,NULL,NULL FROM session_privs-- 
' UNION SELECT granted_role,NULL,NULL FROM user_role_privs-- 
```

Seeing `DBA`, or specific grants such as `CREATE ANY PROCEDURE`, `CREATE ANY JOB`, or execute on the `UTL_` packages, tells you whether command execution, file access, and out-of-band channels are open. With the version, context, and privileges known, the remaining techniques apply with the right expectations.

## References

- Oracle Database SQL Language Reference: SYS_CONTEXT, data dictionary views
- OWASP Testing Guide: Testing for SQL Injection
