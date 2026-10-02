---
title: "PostgreSQL COPY TO PROGRAM and COPY FROM PROGRAM: shell execution with superuser privileges"
description: COPY … PROGRAM runs OS commands when the database role has rights—relevant to chained SQLi only in misconfigured lab systems.
keywords:
  - COPY PROGRAM
  - PostgreSQL RCE
---

# COPY … PROGRAM

## Context

**COPY table TO PROGRAM '…'** and **FROM PROGRAM** invoke the OS from PostgreSQL when allowed. Requires **high privilege** and safe **shell** configuration; not a typical app-user SQLi outcome.

## See also

- [Command execution (parent)](index.md)
