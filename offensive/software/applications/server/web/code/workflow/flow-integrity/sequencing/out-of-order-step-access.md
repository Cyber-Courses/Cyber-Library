---
title: "Out-of-order workflow steps: revisiting intermediate states after later transitions (replay and branch bugs)"
order: 1
description: "Revisiting or repeating intermediate workflow steps after later steps advanced, breaking monotonic progression assumptions in server code."
keywords:
  - workflow
  - out of order
  - replay
---

# Out-of-order access

## Context

Out-of-order access means the user returns to an earlier step after later state has advanced (browser back, a bookmarked mid-flow POST, or a replayed intermediate request) and the server accepts changes that invalidate later commitments such as price, an inventory hold, or an approval. This is distinct from step skip (jumping to the end without context): here the context exists, but the order is wrong.

## Theory

The weak patterns are session flags that only ever increment, no per-form version token, idempotent POST retries that re-apply discounts, and no server-side graph of allowed transitions. When progression is assumed to be monotonic but is not enforced, an earlier step's handler still mutates state the later steps already depended on.

## Practice

### Back-button after price change

- Advance to a review step showing total `T`. Use the browser back button to edit the cart, add items, and return forward. If `T` is not recomputed or locked at commit, the order completes at the stale total; capture the inconsistency in a lab.

### Replay a mid-flow POST

- Capture `POST /checkout/shipping`, then complete payment. Replay the shipping POST to change the address after capture. If the server accepts it, the fulfilled order diverges from the paid-for order.

## Tools

- **Burp Suite**
- **browser**

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
