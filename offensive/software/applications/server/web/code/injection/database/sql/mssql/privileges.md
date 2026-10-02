---
title: "Enumerating and escalating privileges in MSSQL injection"
description: "Reading the SQL Server login's server and database roles, and the common escalation paths: impersonation, trustworthy databases, and sysadmin membership."
keywords:
  - IS_SRVROLEMEMBER
  - sysadmin
  - EXECUTE AS
  - trustworthy database
  - fn_my_permissions
---

# Privileges

The login's roles decide which primitives are reachable, so privilege enumeration follows fingerprinting. The headline check is server-role membership:

```sql
' UNION SELECT IS_SRVROLEMEMBER('sysadmin'),IS_SRVROLEMEMBER('serveradmin'),NULL-- 
```

`sysadmin` is the full administrator: it can enable `xp_cmdshell`, read and write files, dump login hashes, and pivot through linked servers. Note that `serveradmin` is a lesser role that manages server settings and is not equivalent to `sysadmin`; the DBA role to aim for is `sysadmin`. Effective permissions are listed with `fn_my_permissions`:

```sql
' UNION SELECT permission_name,NULL,NULL FROM fn_my_permissions(NULL,'SERVER')-- 
```

When the login is not `sysadmin`, several escalation paths exist. Impersonation grants (`EXECUTE AS LOGIN = 'sa'`) elevate the session when the login holds `IMPERSONATE` on a privileged login. A database marked `TRUSTWORTHY` and owned by a high-privilege login lets a module created in it run with that owner's rights, a route to `sysadmin` from `db_owner`. Membership that can create logins or assign roles (`ALTER ANY LOGIN`, `CONTROL SERVER`) is similarly abusable.

Once `sysadmin` is held or reached, promote a controlled login to be an administrator with:

```sql
'; ALTER SERVER ROLE sysadmin ADD MEMBER [attacker]-- 
```

Knowing the role and its escalation options decides whether to go straight for command execution or to chain an impersonation or trustworthy-database step first.

## References

- Microsoft SQL Server Documentation: IS_SRVROLEMEMBER, fn_my_permissions, EXECUTE AS, TRUSTWORTHY
- OWASP Testing Guide: Testing for SQL Injection
