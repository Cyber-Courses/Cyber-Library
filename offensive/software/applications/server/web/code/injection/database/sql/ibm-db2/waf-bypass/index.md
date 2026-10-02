---
title: "IBM Db2 WAF-oriented obfuscation: hex literals and alternate syntax"
description: Encoding and syntax variants that bypass naive filters—defense remains parameterization and safe SQL APIs.
keywords:
  - WAF bypass
  - Db2
---
# WAF bypass (IBM Db2)

| Topic | Path |
|-------|------|
| Alternate syntax | [Alternate syntax](alternate-syntax.md) |
| Hex encoding | [Hex encoding](hex-encoding.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [IBM Db2 (SQLi)](../index.md)
