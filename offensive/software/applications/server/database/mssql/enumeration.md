---
title: "MSSQL enumeration: instances, logins, and privileges"
description: "Discovering SQL Server instances and versions, enumerating logins, databases, and roles, and establishing exactly which privileges and impersonation rights the current principal holds before attempting execution or escalation."
keywords:
  - MSSQL enumeration
  - SPN
  - sysadmin
  - server roles
  - NetExec
---

# MSSQL enumeration

Before executing anything, map the server and your position in it: the version (to match behaviour), the logins and databases present, and, critically, **what the current principal can already do**. Much of this is readable by the default `public` role.

## Finding instances

```bash
# From the network: service discovery and SPN-based discovery in AD
nmap -p1433 --script ms-sql-info <target>
# MSSQL SPNs (MSSQLSvc/...) in the directory reveal every registered instance
setspn -T example.local -Q MSSQLSvc/*
```

## Logins, databases, and roles

```sql
SELECT @@version;                                  -- version and patch level
SELECT name FROM sys.sql_logins;                   -- SQL logins
SELECT name FROM sys.databases;                    -- databases
SELECT name, type_desc FROM sys.server_principals; -- principals (logins, Windows, roles)
```

## What can the current principal do

```sql
SELECT SUSER_SNAME();                               -- who am I (login)
SELECT IS_SRVROLEMEMBER('sysadmin');                -- 1 if already sysadmin
-- logins I can actually impersonate (tests the IMPERSONATE permission, not mere visibility)
SELECT name FROM sys.server_principals
WHERE type IN ('S','U') AND HAS_PERMS_BY_NAME(name, 'LOGIN', 'IMPERSONATE') = 1;
SELECT * FROM fn_my_permissions(NULL, 'SERVER');    -- my server-level permissions
```

```bash
# NetExec drives the same from the network, including a sysadmin check and query exec
nxc mssql <target> -u user -p pass --local-auth -q "SELECT IS_SRVROLEMEMBER('sysadmin')"
```

## Exploitation notes

- The first question is always **`IS_SRVROLEMEMBER('sysadmin')`**: if you are already sysadmin, go straight to [command execution](command-execution.md); if not, enumerate [impersonation](impersonation.md) targets and [linked servers](linked-servers.md).
- `public` can usually read `sys.databases`, `sys.server_principals`, and call the coercion procedures, so enumeration rarely needs special rights.
- Note **TRUSTWORTHY** databases and database ownership here (`SELECT name, is_trustworthy_on FROM sys.databases`), since they feed the escalation chains.
- Windows-authenticated access matters: a [relayed](coercion-and-relay.md) or domain login may map to a more privileged principal than a SQL login.

## Tools

- **NetExec `mssql`** (`-q`/`-x`, `--local-auth`): network enumeration, sysadmin check, query execution.
- **Impacket `mssqlclient.py`**: interactive authenticated enumeration.
- **nmap `ms-sql-*`**: unauthenticated instance and version discovery.

## References

- [NetExec: MSSQL methodology](https://www.netexec.wiki/mssql-protocol/enumeration)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [Microsoft: sys.server_principals](https://learn.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-server-principals-transact-sql)
