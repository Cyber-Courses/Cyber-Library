---
title: "Reversal, chargeback, and refund workflow abuse: race with settlement and duplicate credits"
description: Compensating transactions that credit accounts before debits settle, or that allow multiple refunds for one charge.
keywords:
  - workflow
  - refund
---

# Reversal workflow abuse

## Context

**Refund** APIs may not be **idempotent**; **parallel** refund and **capture** requests can **double-credit** when ledger updates are not atomic. **Chargeback** webhooks may be **replayed** if event ids are not stored.

## See also

- [Sequencing (parent)](index.md)
- [Parallel effects](../../parallel-effects/index.md)
