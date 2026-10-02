---
title: "Oracle UTL_INADDR and DNS-style out-of-band channels"
description: Host resolution functions that can encode data in DNS lookups—egress and package grants.
keywords:
  - UTL_INADDR.get_host_name
  - DNS exfiltration
---
# UTL INADDR (Oracle)

**UTL_INADDR** resolves **hostnames** and can be abused for **DNS**-style **OOB** when labels embed **query-derived** strings and the resolver reaches the internet.

## Context

`UTL_INADDR.GET_HOST_NAME` forces lookups; embed subquery output in the hostname so each lookup leaks bytes via DNS to your zone.
## Technique

Resolver must reach the internet or internal DNS that forwards to you.
## Practice

- Label length limits apply—design chunking accordingly.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Out of band (Oracle)](index.md)
