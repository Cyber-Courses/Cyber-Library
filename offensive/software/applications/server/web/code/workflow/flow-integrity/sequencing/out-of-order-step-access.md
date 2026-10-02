---
title: "Out-of-order workflow steps: revisiting intermediate states after later transitions (replay and branch bugs)"
description: Revisiting or repeating intermediate workflow steps after later steps advanced, breaking monotonic progression assumptions in server code.
keywords:
  - workflow
  - out of order
  - replay
---

# Out-of-order access

## Context

**Out-of-order** access means the **user** returns to an **earlier** **step** **after** **later** **state** **advanced**, **browser** **back**, **bookmarked** **mid**-**flow** **POST**, or **replayed** **intermediate** **request**, and the **server** **accepts** **changes** that **invalidate** **later** **commitments** (price, inventory hold, approval). Distinct from **step skip** (jump to **end** **without** **context**): here **context** **exists** but **order** **is** **wrong**.

## Theory

Weak patterns: **session** **flags** that **only** **increment**, **no** **version** **token** on **each** **form**, **idempotent** **POST** **retries** that **re**-**apply** **discounts**, **and** **no** **server**-**side** **graph** of **allowed** **transitions**.

## Practice

### Back-button after price change

- Advance to a **review** step showing **total** **T**. Use **back** to **edit** **cart**, **add** **items**, **return** **forward**, if **T** **is** **not** **recomputed** or **locked**, capture the **inconsistency** in a **lab**.

### Replay mid-flow POST

- Capture `POST /checkout/shipping`. Complete **payment**. **Replay** **shipping** **POST** to **change** **address** **after** **capture** if the **server** **accepts** it.

## Tools

- **Burp Suite**
- **browser**
