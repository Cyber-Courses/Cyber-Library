---
title: "Flow integrity in web applications: sequencing versus UI, record, and session state alignment"
description: Workflow coherence—sequencing of steps versus alignment of state across clients, records, and sessions.
keywords:
  - flow integrity
  - business logic
  - workflow
---

# Flow integrity

**Flow integrity** asks whether the **story** of a workflow is coherent before you analyze races or numeric tampering. The planner splits it into **sequencing** (order, approvals, reversals) and **state alignment** (what the UI, database row, and session each believe).

## Child topics

- **[Sequencing](sequencing/index.md)** — Step skip, out-of-order access, approval bypass, parameter omission, entitlement renewal, reversal workflows.
- **[State alignment](state-alignment/index.md)** — UI vs record vs session checkpoints.
- **[Parallel effects](../parallel-effects/index.md)** — Races, double spend, callback replay (sibling under workflow).

**Trust boundaries** for numeric and business parameters: **[Trust boundaries](../trust-boundaries/index.md)**.

## See also

- [Workflow (parent)](../index.md)
