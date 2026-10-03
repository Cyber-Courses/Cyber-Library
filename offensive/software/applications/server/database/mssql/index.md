---
title: "Microsoft SQL Server"
description: "The offensive surface of Microsoft SQL Server: authenticating and enumerating, executing OS commands with xp_cmdshell and other sinks, moving laterally over linked servers, escalating through impersonation and TRUSTWORTHY ownership chains, and coercing the service account for NTLM relay."
keywords:
  - MSSQL
  - SQL Server
  - xp_cmdshell
  - linked servers
  - NTLM relay
---

# Microsoft SQL Server

Microsoft SQL Server (MSSQL, port 1433) is the most productive database target in a Windows estate. It authenticates both SQL logins and **Windows/domain accounts**, its service runs as a privileged account that speaks **NTLM**, and it ships with stored procedures that **run operating-system commands**, **reach other servers**, and **touch the filesystem**. A single low-privilege login often chains to SYSTEM on the host or to Domain Admin.

## Why MSSQL matters

- It is **AD-integrated**: domain users can be database principals, and the service account's authentication can be coerced and [relayed](coercion-and-relay.md).
- It executes code: `xp_cmdshell` and several other sinks give OS command execution as the service account.
- It is **networked to other servers** through linked servers, so one instance pivots to many.
- Its privilege model (impersonation, database ownership, TRUSTWORTHY) has well-worn **escalation** chains from `public` to `sysadmin`.

## Pages

- **[Enumeration](enumeration.md)**: finding instances, versions, logins, databases, and the privileges you hold.
- **[Access](access.md)**: authenticating as a SQL or domain login and the roles that matter.
- **[Command execution](command-execution.md)**: xp_cmdshell, OLE automation, and CLR for OS commands.
- **[Linked servers](linked-servers.md)**: querying and executing across server trust links.
- **[Impersonation](impersonation.md)**: EXECUTE AS and TRUSTWORTHY ownership chains to sysadmin.
- **[Coercion and relay](coercion-and-relay.md)**: xp_dirtree/xp_fileexist to capture or relay the service account.

## References

- [HackTricks: pentesting MSSQL (1433)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [NetExec: MSSQL protocol and modules](https://www.netexec.wiki/mssql-protocol)
- [Impacket mssqlclient](https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py)
