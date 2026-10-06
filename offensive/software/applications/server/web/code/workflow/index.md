---
title: "Web application workflow testing: sequencing, concurrency, races, and business-parameter integrity"
order: 4
description: "Business-logic and state integrity in multi-step and multi-service web flows: sequencing, concurrency, and parameter integrity."
keywords:
  - business logic
  - workflow security
  - TOCTOU
  - idempotency
---

# Workflow

**Workflow** covers **sequencing** (order of steps and gates), **parallel effects** (races, double spend, replayed callbacks), and **trust** in numeric or business parameters (amounts, quotas) across long flows. It assumes access control and injection primitives are understood at sibling nodes.

## Structure (library map)

- **[Flow integrity](flow-integrity/index.md)**: Sequencing vs state alignment across UI, records, and sessions.
- **[Parallel effects](parallel-effects/index.md)**: Concurrency and integration handoff (TOCTOU, replay, idempotency misuse).
- **[Trust boundaries](trust-boundaries/index.md)**: Numeric and business-parameter integrity across steps.

