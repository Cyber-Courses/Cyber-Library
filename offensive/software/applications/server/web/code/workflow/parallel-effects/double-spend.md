---
title: "Double spend in web applications: coupons, wallet credits, one-time tokens, and inventory reservations"
description: "Applying the same credit, coupon, refund token, or inventory reservation more than once through parallel or retried requests."
keywords:
  - double spend
  - coupon abuse
---

# Double spend

## Context

“Double spend” in web testing means reusing a single authorization artifact (promo code, wallet balance, one-time token) across two successful commits. It differs from payment-network double-spend; here the bug is application-level ledger or coupon logic.

## Theory

Fails when redemption is not atomic with the debit, or when idempotency keys are missing for retried submits.

## Practice

- Capture a successful redemption request; replay it in parallel from two sessions or two TCP connections before the first response completes.

## Tools

- **Burp Suite**
- **OWASP ZAP**