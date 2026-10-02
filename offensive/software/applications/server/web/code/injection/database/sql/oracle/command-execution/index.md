---
title: "Oracle command execution via SQL: Java in the database and policy"
description: Java-related execution surfaces in Oracle Database and why application accounts should not hold Java privilege grants or job creation rights.
keywords:
  - dbms_java
  - Java Oracle
---
# Command execution (Oracle)

| Topic | Path |
|-------|------|
| Java class | [Java class](java-class.md) |
| Java execution | [Java execution](java-execution.md) |

Oracle can host **Java** in the database; combined with weak grants, this expands post-injection **execution** risk. Many deployments **disable** or tightly scope Java and rely on external job runners instead.

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](../index.md)
- [File manipulation](../file-manipulation/index.md)
