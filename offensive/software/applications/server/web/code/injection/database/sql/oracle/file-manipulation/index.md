---
title: "Oracle file read and write: UTL_FILE, DBMS_LOB, and scheduler jobs"
description: Oracle file read/write and job surfaces—UTL_FILE, DBMS_LOB, DBMS_SCHEDULER—for data theft and execution chains from SQL injection.
keywords:
  - UTL_FILE
  - DBMS_LOB
  - DBMS_SCHEDULER
---
# File manipulation (Oracle)

| Topic | Path |
|-------|------|
| DBMS SCHEDULER jobs | [DBMS SCHEDULER jobs](dbms-scheduler-jobs.md) |
| Package OS command | [Package OS command](package-os-command.md) |
| Read file | [Read file](read-file.md) |
| Write file | [Write file](write-file.md) |

**Directory objects** and **`UTL_FILE`** / **`DBMS_LOB`** grants map Oracle to the **host filesystem** where allowed. **Scheduler** can run **external** jobs when configuration permits.

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](../index.md)
