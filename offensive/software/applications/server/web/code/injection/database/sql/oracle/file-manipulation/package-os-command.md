---
title: "Oracle package OS command: scheduler and external job abuse"
description: OS-oriented abuse paths through scheduler and external job features—overlap with file and command execution topics.
keywords:
  - external_job
  - dbms_scheduler
---
# Package OS command (Oracle)

Some **packages** and **job types** bridge SQL to **host** behavior. Treat these as **admin-tier** capabilities, not application defaults.

## Context

Some packaged procedures shell out or spawn jobs—vendor-specific; hunt in `ALL_PROCEDURES` for suspicious names.
## Technique

Often overlaps scheduler external job setup.
## Practice

- Read package bodies with `ALL_SOURCE` when accessible.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [File manipulation (Oracle)](index.md)
- [Command execution](../command-execution/index.md)
