---
title: "Parallel effects in web workflows: races, double spend, webhook replay, and idempotency misuse"
description: Concurrency and integration handoff in workflows, races, double spend, replayed callbacks, and idempotency misuse.
keywords:
  - TOCTOU
  - race condition
  - webhook replay
  - idempotency
---

# Parallel effects

Parallel effects cover what happens when two or more requests touch the same workflow state, or when asynchronous callbacks arrive out of order. They sit beside [sequencing](../flow-integrity/sequencing/index.md) (order of user-facing steps) and [state alignment](../flow-integrity/state-alignment/index.md) (what each layer believes).

## Concurrency

| Page | Focus |
|------|--------|
| [TOCTOU in workflows](toctou.md) | Check-then-act gaps across requests |
| [Double spend](double-spend.md) | Spending the same balance, coupon, or credit twice |
| [Parallel requests](parallel-requests.md) | Intentional request fan-out against weak locks |

## Integration handoff

| Page | Focus |
|------|--------|
| [Webhook replay](webhook-replay.md) | Duplicate delivery of signed or unsigned callbacks |
| [Out-of-order callback](out-of-order-callback.md) | Payment or provisioning events applied in wrong order |
| [Idempotency key misuse](idempotency-key-misuse.md) | Keys ignored, scoped wrong, or reset on retry |
