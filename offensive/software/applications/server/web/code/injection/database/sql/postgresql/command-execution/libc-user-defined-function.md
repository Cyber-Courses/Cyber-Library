---
title: "PostgreSQL user-defined functions and libc for command execution (historical patterns)"
description: CREATE FUNCTION … LANGUAGE C loading shared libraries—PostgreSQL post-exploitation and privileged SQL contexts.
keywords:
  - PostgreSQL UDF
  - libc
---

# libc / UDF

## Context

Loading **C** functions from **shared libraries** to run **system** commands is an older escalation path on mis-hardened clusters. Modern deployments restrict **dynamic** library loading and **file** placement.

## See also

- [Command execution (parent)](index.md)
