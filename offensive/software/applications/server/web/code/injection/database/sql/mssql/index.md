---
title: "Microsoft SQL Server SQL injection: T-SQL patterns, stacked queries, and linked-server abuse"
description: Microsoft SQL Server–specific SQL injection patterns, union, blind, error-based, time-based, file and command channels, and out-of-band exfiltration in application code.
keywords:
  - MSSQL SQL injection
  - SQL Server
  - T-SQL
  - stacked queries
---
# Microsoft SQL Server (SQLi)

**Microsoft SQL Server** exposes a large T-SQL surface: `UNION`, blind inference with string functions, error-based casting, `WAITFOR DELAY`, stacked batches when the driver allows multiple statements, extended procedures, and linked-server paths. The layout below follows the **Library Structure** topic tree for MSSQL (structure topic id 1962).

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## Core techniques

| Topic | Path |
|-------|------|
| Union-based | [Union-based](union-based.md), column typing, `NULL` padding, `TOP` / `OFFSET-FETCH` |
| Blind | [Blind](blind.md), `SUBSTRING`, `ASCII`, inference on row counts |
| Time-based | [Time-based](time-based.md), `WAITFOR DELAY`, conditional timing |
| Error-based | [Error-based](error-based.md), type conversion and cast-driven errors |
| Stacked query | [Stacked query](stacked-query.md), `;`, `EXEC`, driver-dependent batching |

## Data, trust, and execution

| Topic | Path |
|-------|------|
| Database credentials | [Database credentials](database-credentials.md), login metadata and hash exposure |
| File manipulation | [File manipulation](file-manipulation.md), `BULK`, `OPENROWSET`, scripting surfaces |
| Command execution | [Command execution](command-execution.md), `xp_cmdshell`, automation and policy context |
| Out of band | [Out of band](out-of-band/index.md), DNS and UNC-style network callbacks |
| Privileges | [Privileges](privileges/index.md), effective permissions and role membership |
| Trusted links | [Trusted links](trusted-links.md), linked servers and `OPENQUERY` |
