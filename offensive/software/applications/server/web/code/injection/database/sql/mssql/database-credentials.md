---
title: "Dumping SQL Server login password hashes"
description: "Extracting SQL Server login password hashes from sys.sql_logins for offline cracking, and the older syslogins and sysxlogins sources."
keywords:
  - password hash dump
  - sys.sql_logins
  - password_hash
  - syslogins
  - LOGINPROPERTY
---

# Database credentials

SQL Server stores the hashes of SQL-authenticated logins, and a high-privilege injection can read them for offline cracking, which may recover the `sa` password or credentials reused elsewhere.

On SQL Server 2012 and later the hashes are in `sys.sql_logins.password_hash` (a `varbinary`), converted to hex for transport:

```sql
' UNION SELECT name,master.dbo.fn_varbintohexstr(password_hash),NULL FROM sys.sql_logins-- 
```

The hex string (beginning `0x0200...` for the current hashing scheme) is cracked offline, for example with hashcat's SQL Server 2012/2014 mode. On older servers the sources differ: `sys.syslogins` and `master..syslogins` expose a `password` column on 2005/2008, and `master..sysxlogins` on SQL Server 2000. `LOGINPROPERTY(name,'PasswordHash')` also returns the hash where direct catalog reads are blocked:

```sql
' UNION SELECT name,master.dbo.fn_varbintohexstr(CAST(LOGINPROPERTY(name,'PasswordHash') AS varbinary(256))),NULL FROM sys.sql_logins-- 
```

Reading these requires `CONTROL SERVER` or `sysadmin` (the password hash columns are protected), so confirm the role first. Windows-authenticated logins have no stored hash here; this route only recovers SQL-authentication credentials.

## Tools

- hashcat (SQL Server hash modes)

## References

- Microsoft SQL Server Documentation: sys.sql_logins, LOGINPROPERTY
- OWASP Testing Guide: Testing for SQL Injection
