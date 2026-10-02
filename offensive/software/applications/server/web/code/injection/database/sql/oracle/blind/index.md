---
title: "Oracle blind SQL injection: boolean and time-based inference"
description: Blind SQL injection in Oracle—CASE, SUBSTR, DUAL, and delay primitives when errors and union output are not visible.
keywords:
  - Oracle blind SQLi
  - DUAL
---
# Blind (Oracle)

| Topic | Path |
|-------|------|
| Boolean-based | [Boolean-based](boolean-based.md) |
| Time-based | [Time-based](time-based.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](../index.md)
