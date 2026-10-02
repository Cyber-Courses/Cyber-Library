---
title: "PostgreSQL boolean blind SQLi using pg_backend_pid(): process identity in predicates"
description: Rare inference channel comparing backend process identifiers when other secrets are hard to reach—mostly a lab curiosity.
keywords:
  - PostgreSQL SQL injection
  - pg_backend_pid
---

# pg_backend_pid

## Context

**pg_backend_pid()** returns the session’s backend PID. Equality checks can form **boolean** predicates when other columns are unavailable. Maps to Library Structure **Boolean with pg backend pid**.

## See also

- [Boolean based (parent)](index.md)
