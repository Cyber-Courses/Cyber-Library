---
title: "SQLite enumeration: sqlite_master, PRAGMA, and version fingerprinting"
description: Schema and environment discovery in SQLite—master tables, pragma_table_info, and compile options.
keywords:
  - sqlite_master
  - pragma_table_info
  - sqlite_version
---
# Enumeration techniques (SQLite)

| Topic | Path |
|-------|------|
| Column and data type discovery | [Column and data type discovery](column-and-data-type-discovery.md) |
| Schema and table discovery | [Schema and table discovery](schema-and-table-discovery.md) |
| Version and environment detection | [Version and environment detection](version-and-environment-detection.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [SQLite (SQLi)](../index.md)
