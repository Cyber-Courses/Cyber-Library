---
title: "MSSQL DNS exfiltration in SQL injection: privileged metadata and DNS labels"
description: How attackers may encode data in DNS queries when SQL Server functions or audit interfaces can build hostnames—defensive egress and privilege controls.
keywords:
  - DNS exfiltration
  - VIEW SERVER STATE
  - MSSQL
---
# DNS exfiltration (MSSQL)

**DNS exfiltration** builds a **subdomain** or **hostname** string from query results so a resolver lookup leaks data to an attacker-controlled zone. Practical paths depend on **available functions**, **permissions** (for example extended views of events or traces), and **outbound DNS** policy.

## Context

DNS OOB splits data across subdomain labels of a zone you control; the resolver issues lookups that hit your authoritative server.
## Technique

Concatenate hex chunks into `master..fn_varbintohexstr`-style labels or use error-to-string pipelines that fit hostname rules.
## Practice

- Keep labels under DNS length limits; chunk aggressively.
- Requires functions that perform name resolution or error paths that trigger DNS—validate in staging.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Out of band (MSSQL)](index.md)
