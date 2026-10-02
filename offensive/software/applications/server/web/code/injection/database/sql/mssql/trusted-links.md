---
title: "MSSQL trusted links and linked servers: OPENQUERY and lateral movement"
description: Linked servers in SQL Server as a trust boundary—how injection may pivot when OPENQUERY or EXECUTE AT is available.
keywords:
  - linked servers
  - OPENQUERY
  - sp_linkedservers
---
# Trusted links (MSSQL)

**Linked servers** let one SQL Server instance query another. If an application login can **execute** distributed queries or run procedures **as** a privileged link, injection may **pivot** to other hosts in the trust graph.

## Context

Linked servers let you `EXEC ('...') AT [link]` or `OPENQUERY` to hit remote SQL instances with stored credentials.
## Technique

Enumerate **`sys.servers`**, **`sp_linkedservers`**, then test `OPENQUERY` for code execution on the remote hop.
## Practice

- Map **login mappings**—misconfigured `rpc out` can widen impact.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](index.md)
- [Privileges](privileges/index.md)
