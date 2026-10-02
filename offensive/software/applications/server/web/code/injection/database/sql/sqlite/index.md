---
title: "SQLite SQL injection: embedded databases, PRAGMA, and extension loading"
description: SQLite-specific SQL injection—blind inference, sqlite_master introspection, file-oriented abuse, and load_extension risk in application deployments.
keywords:
  - SQLite SQL injection
  - sqlite_master
  - load_extension
---
# SQLite (SQLi)

**SQLite** is often **embedded** in mobile apps, desktop software, and edge services. Features include **`sqlite_master`** introspection, **`PRAGMA`**, **`ATTACH`**, and optional **`load_extension`**. The layout below follows the **Library Structure** topic tree for SQLite (structure topic id 2010).

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## Techniques

| Topic | Path |
|-------|------|
| Blind | [Blind](blind/index.md) |
| Enumeration techniques | [Enumeration techniques](enumeration-techniques/index.md) |
| Error-based | [Error-based](error-based.md) |
| Evasion techniques | [Evasion techniques](evasion-techniques.md) |
| File manipulation | [File manipulation](file-manipulation/index.md) |
| Remote code execution | [Remote code execution](remote-code-execution/index.md) |

## See also

- [SQL injection (parent)](../index.md)
