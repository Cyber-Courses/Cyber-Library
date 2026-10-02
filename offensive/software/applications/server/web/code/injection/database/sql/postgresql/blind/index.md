---
title: "Blind SQL injection (PostgreSQL): boolean channels without visible query errors"
description: Inferring data from true/false application responses using PostgreSQL predicates and substring tests.
keywords:
  - PostgreSQL SQL injection
  - blind SQLi
---

# Blind (PostgreSQL)

Blind SQLi uses **boolean** or **timing** channels when union or error output is not visible. The Library Structure groups boolean techniques under **Boolean based**.

## Pages

| Page | Focus |
|------|--------|
| [Boolean based](boolean-based/index.md) | CASE, substring, NULLIF, pg_backend_pid |
