---
title: "PostgreSQL file read and write via SQL: COPY, large objects, and pg_read_file"
description: File-system interaction primitives exposed through SQL when roles have dangerous privileges, COPY path, lo_*, pg_read_file.
keywords:
  - PostgreSQL file read
  - COPY
---

# File manipulation

## Context

PostgreSQL exposes **file-oriented** primitives, **`COPY FROM/TO`**, **large objects** (`lo_*`), **`pg_read_file`**, **`pg_ls_dir`**, when the session has the right **roles** and **`pg_hba` / OS** layout allows the server process to touch those paths. In **application** SQLi, you only win this when the **effective user** is over-permissioned; most web roles are locked down, so confirm **privileges** before investing in file chains.

## Technique

Chain file reads into **blind** or **error-based** extraction, or write **COPY** / **LO** paths toward **webshell** or **cron** locations only after you map **data directory**, **log**, and **UMASK** layout. **`COPY ... PROGRAM`** (where available) is the OS-command class, treat like command execution, not “read file” alone.

## Practice

- Enumerate role attrs: `rolsuper`, `rolcreaterole`, membership in **`pg_read_server_files`** / **`pg_write_server_files`** (names vary by version).
- Prefer **`pg_read_file`** for short reads; use **`COPY`** for bulk exfil when path and perms align.
- If file primitives fail, pivot to **`COPY TO PROGRAM`**, extensions, or **session** tokens from the DB layer.

## Tools

- **sqlmap** (file-read options where applicable)
- **`psql`** for interactive confirmation
- **Burp Suite** for blind/error channel tuning

## Scope

Authorized assessments and isolated labs only.
