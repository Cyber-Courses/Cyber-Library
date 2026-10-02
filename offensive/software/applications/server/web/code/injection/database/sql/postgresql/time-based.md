---
title: "Time-based blind SQL injection in PostgreSQL: pg_sleep and conditional delays"
description: Timing side channels using pg_sleep(), statement_timeout interaction, and CASE-wrapped delays.
keywords:
  - PostgreSQL SQL injection
  - pg_sleep
---

# Time based

## Context

Library Structure places **Time Based** as a single PostgreSQL topic. Combine **pg_sleep** with predicates for **bit** extraction when **boolean** responses are too noisy.

## See also

- [PostgreSQL (parent)](index.md)
- [Blind](blind/index.md)
