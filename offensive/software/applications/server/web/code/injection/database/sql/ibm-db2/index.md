---
title: "IBM Db2 SQL injection: SYSCAT, XML helpers, and administrative interfaces"
description: IBM Db2–specific SQL injection patterns, blind inference, catalog enumeration, error-based XML helpers, and WAF-oriented obfuscation.
keywords:
  - Db2 SQL injection
  - SYSCAT
  - SYSIBM
---
# IBM Db2 (SQLi)

**IBM Db2** exposes rich **catalog** views (**`SYSCAT`**, **`SYSIBM`**) and XML-oriented helpers used in **error-based** extraction. The layout below follows the **Library Structure** topic tree for IBM DB2 (structure topic id 2026).

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## Techniques

| Topic | Path |
|-------|------|
| Blind | [Blind](blind/index.md) |
| Command execution | [Command execution](command-execution.md) |
| DIOS dump in one shot | [DIOS dump in one shot](dios-dump-in-one-shot.md) |
| Enumeration techniques | [Enumeration techniques](enumeration-techniques/index.md) |
| Error-based | [Error-based](error-based.md) |
| WAF bypass | [WAF bypass](waf-bypass/index.md) |
