---
title: "IBM Db2 enumeration: SYSCAT, SYSIBM, and session metadata"
description: Schema and environment discovery in Db2, catalog views and session identifiers for SQL injection reconnaissance.
keywords:
  - SYSCAT
  - SYSIBM
---
# Enumeration techniques (IBM Db2)

| Topic | Path |
|-------|------|
| Schema and metadata discovery | [Schema and metadata discovery](schema-and-metadata-discovery.md) |
| Session and user information | [Session and user information](session-and-user-information.md) |
| Version and environment detection | [Version and environment detection](version-and-environment-detection.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
