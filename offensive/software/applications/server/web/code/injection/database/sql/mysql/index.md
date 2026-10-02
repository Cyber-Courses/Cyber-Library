---
title: "MySQL SQL injection: techniques, payloads, and data extraction"
description: "Exploiting SQL injection against MySQL and MariaDB: comment syntax, information_schema enumeration, and the union, error, blind, time, out-of-band, and file-access techniques."
keywords:
  - MySQL SQL injection
  - MariaDB injection
  - information_schema
  - union based injection
  - error based injection
  - MySQL file privileges
---

# MySQL

MySQL (and the compatible MariaDB fork) is the database behind a large share of injectable web applications, so its dialect is worth knowing precisely. Several traits shape how injection plays out against it.

Comments are `-- ` (the trailing space is required), `#`, and `/* */`. Versioned comments `/*! ... */` execute their contents only on MySQL at or above an embedded version number, which doubles as a filter-evasion trick. String literals concatenate with `CONCAT()` rather than `||` (the `||` operator means logical OR unless `PIPES_AS_CONCAT` is set), and hex literals such as `0x7573657273` stand in for quoted strings when quotes are filtered.

Schema metadata lives in `information_schema` (databases in `SCHEMATA`, tables in `TABLES`, columns in `COLUMNS`), which is the backbone of blind and union extraction. `database()`, `user()`, and `@@version` identify the current context. Stacked queries are usually unavailable: the common PHP drivers (`mysqli_query`, PDO with emulation) send one statement per call, so techniques that depend on a second `;`-separated statement rarely work here, unlike MSSQL or PostgreSQL.

File access is gated. `LOAD_FILE()` and `SELECT ... INTO OUTFILE/DUMPFILE` require the `FILE` privilege, and the `secure_file_priv` system variable restricts (or disables) the directories they can touch. These defaults decide whether file read, web-shell write, and UDF command execution are reachable.

## Techniques

- **[Union-based](union-based/index.md)**: append a `UNION SELECT` to pull data into the visible response.
- **[Error-based](error-based/index.md)**: force query output into a reflected error message.
- **[Boolean blind](blind/index.md)**: infer data one bit at a time from true/false response differences.
- **[Time-based](time-based/index.md)**: infer data from conditional response delays.
- **[Out-of-band](out-of-band/index.md)**: exfiltrate over DNS or SMB when no channel is reflected.
- **[File read](read-file.md)**: read server files with `LOAD_FILE()`.
- **[Command execution](command-execution/index.md)**: write web shells or load a UDF for OS commands.
- **[WAF bypass](waf-bypass/index.md)**: reach `information_schema`, `version()`, and keywords past filters.

## References

- MySQL Reference Manual: `information_schema` tables and string functions
- OWASP Testing Guide: Testing for SQL Injection
