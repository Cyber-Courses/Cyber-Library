---
title: "Oracle TNS listener and SID enumeration: network exposure"
description: TNS listener reconnaissance, port, service names, SID brute force, and banner-style fingerprinting before SQL injection work.
keywords:
  - TNS listener
  - SID enumeration
  - Oracle 1521
---
# Listener and SID enumeration (Oracle)

The **TNS listener** advertises database connectivity on the network (classically **TCP 1521**). **Recon** here is **version** probes, **service name** / **SID** guessing, and mapping what you can hit **before** you spend time on app-layer SQLi, useful for scoping credentials and lateral moves.

## Context

Off-network recon: identify listener port (often 1521), service names/SIDs, and patch level from banners before touching SQLi.
## Technique

Use `lsnrctl`, `nmap` scripts, or Oracle SQL Developer connectivity tests in authorized scope.
## Practice

- SID brute force is noisy, coordinate with client detection teams.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
