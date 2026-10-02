---
title: "SQLite version and environment: sqlite_version and compile options"
description: Fingerprinting SQLite builds with sqlite_version() and PRAGMA compile_options for patch awareness.
keywords:
  - sqlite_version
  - compile_options
---
# Version and environment detection (SQLite)

**`sqlite_version()`** returns the **library** version string. **`PRAGMA compile_options`** lists **feature** flags relevant to **security** (for example extension loading).

## Context

`sqlite_version()` and `pragma compile_options` fingerprint build flags, `load_extension` presence matters for RCE chains.
## Technique

Check whether `ENABLE_LOAD_EXTENSION` is on in deployment.
## Practice

- Use compile options to plan extension vs file-only paths.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
