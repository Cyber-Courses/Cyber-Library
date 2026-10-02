---
title: "MySQL and MariaDB SQL injection: union, blind, error-based, time-based, and dialect-specific patterns"
description: MySQL-specific SQL injection patterns, union, blind, error-based, time-based, file read, and WAF-oriented variants in application code.
keywords:
  - MySQL SQL injection
  - MariaDB
  - union SQLi
---

# MySQL (SQLi)

**MySQL** (and **MariaDB**-compatible) deployments share a large feature surface: `UNION`, `LOAD_FILE`, `INTO OUTFILE` / `DUMPFILE` where allowed, error-based channels, blind boolean and time delays, and out-of-band channels when the stack and network permit. The layout below follows the **Library Structure** topic tree for MySQL (structure topic id 1906).

## Core techniques

| Topic | Path |
|-------|------|
| Union-based | [Union-based](union-based/index.md), column alignment, `information_schema`, extraction without metadata |
| Blind | [Blind](blind/index.md), `LIKE`, `REGEXP`, `IF`, substring tests |
| Time-based | [Time-based](time-based/index.md), `SLEEP` and conditional delays in subselects |
| Error-based | [Error-based](error-based/index.md), `UPDATEXML`, `EXTRACTVALUE`, `GROUP BY` error channels |

## Library Structure siblings

| Topic | Path |
|-------|------|
| Command execution | [Command execution](command-execution/index.md), `OUTFILE`, `DUMPFILE`, UDF |
| Out of band | [Out of band](out-of-band/index.md), DNS, UNC / NTLM context |
| WAF bypass | [WAF bypass](waf-bypass/index.md), metadata alternatives, encodings, comments |
| DIOS (one-shot) | [DIOS dump in one shot](dios-dump-in-one-shot.md) |
| INSERT-based | [INSERT-based SQLi](insert-based-sql-injection.md) |
| Read file | [Read file (LOAD_FILE)](read-file-load-file.md) |
| Truncation | [Truncation](truncation.md) |
