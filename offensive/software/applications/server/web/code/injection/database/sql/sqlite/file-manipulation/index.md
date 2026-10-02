---
title: "SQLite file read and write: ATTACH, export, and .dump surfaces"
description: File-oriented behavior in SQLite, reading databases from disk, attaching databases, and writing content via export APIs.
keywords:
  - ATTACH DATABASE
  - .dump
---
# File manipulation (SQLite)

| Topic | Path |
|-------|------|
| Read file | [Read file](read-file.md) |
| Write file | [Write file](write-file.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
