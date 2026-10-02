---
title: "SQLite remote code execution: load_extension and ATTACH abuse"
description: High-impact SQLite primitives, ATTACH DATABASE and load_extension, for code execution and cross-database reads during SQL injection.
keywords:
  - load_extension
  - ATTACH DATABASE
---
# Remote code execution (SQLite)

| Topic | Path |
|-------|------|
| Attach database | [Attach database](attach-database.md) |
| Load extension | [Load extension](load-extension.md) |

**`load_extension`** loads **native** code (**`.so`** / **`.dll`**). **`ATTACH`** can chain into **unexpected** DB files when paths are attacker-controlled. Many deployments **disable** extension loading.

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
