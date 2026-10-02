---
title: "Oracle session and user enumeration: SYS_CONTEXT and USERENV"
description: Oracle session metadata—current user, schema, host, and IP—for orienting SQLi chains and pivot planning.
keywords:
  - SYS_CONTEXT
  - USERENV
  - CURRENT_USER
---
# Session and user enumeration (Oracle)

**`USER`**, **`SYS_CONTEXT('USERENV', ...)`**, and related calls expose **who** you are, **where** the session thinks it is, and **host/IP** hints. Use that to choose **OOB** labels, **file** paths, and whether **`UTL_*`** packages are worth chasing.

## Context

Orientation queries: `USER`, `SYS_CONTEXT('USERENV','SESSION_USER')`, host/IP context for pivot planning.
## Technique

Use results to pick OOB domains, file paths, or credential reuse hypotheses.
## Practice

- Combine with `ALL_USERS` / `DBA_USERS` when privileges allow hash or profile review.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](index.md)
