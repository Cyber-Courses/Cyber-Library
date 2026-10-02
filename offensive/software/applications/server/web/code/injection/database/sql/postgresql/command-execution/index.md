---
title: "PostgreSQL SQL injection leading to OS command execution: COPY PROGRAM and UDFs"
description: High-privilege database features sometimes reachable through injection chains—lab and defense context only.
keywords:
  - PostgreSQL
  - COPY PROGRAM
  - UDF
---

# Command execution (PostgreSQL)

These topics mirror **Library Structure** under PostgreSQL → **Command Execution**. They assume **superuser** or dangerous defaults; in **application** SQLi the path is often blocked by privileges—document for completeness, not as a universal exploit.

## Pages

| Page | Focus |
|------|--------|
| [COPY … PROGRAM](copy-from-program.md) | `COPY … TO/FROM PROGRAM` |
| [libc / UDF](libc-user-defined-function.md) | Shared-library UDF patterns |

## See also

- [PostgreSQL (parent)](../index.md)
