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

- Prepare a single unconsumed redemption (a fresh coupon, token, or reservation that has never been applied), capture the redemption request without sending it, then dispatch several identical copies concurrently (Turbo Intruder single-packet attack, or parallel connections). The goal is for all copies to pass the "is it still valid?" check before any one of them marks the artifact consumed. Replaying an already-completed redemption only tests post-commit reuse; the race requires the duplicates to arrive while the artifact is still unconsumed.

## Tools

- **Burp Suite**
- **OWASP ZAP**

## References

- PortSwigger Web Security Academy: Race conditions
- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic