---
title: "Oracle union-based SQL injection: DUAL, ROWNUM, and type alignment"
description: Union-based extraction in Oracle—column discovery with ORDER BY, NULL padding, and DUAL for probes.
keywords:
  - UNION SELECT
  - DUAL
  - ROWNUM
---
# Union-based (Oracle)

| Topic | Path |
|-------|------|
| Banner disclosure via UNION | [Banner disclosure via UNION](banner-disclosure-via-union.md) |
| Column number discovery | [Column number discovery](column-number-discovery.md) |
| Credential extraction | [Credential extraction](credential-extraction.md) |
| Data type matching | [Data type matching](data-type-matching.md) |
| Dual table usage | [Dual table usage](dual-table-usage.md) |
| Null padding | [Null padding](null-padding.md) |
| Row filtering | [Row filtering](row-filtering.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](../index.md)
