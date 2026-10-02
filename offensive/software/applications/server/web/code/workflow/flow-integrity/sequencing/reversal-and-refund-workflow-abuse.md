---
title: "Reversal, chargeback, and refund workflow abuse: race with settlement and duplicate credits"
description: "Compensating transactions that credit accounts before debits settle, or that allow multiple refunds for one charge."
keywords:
  - workflow
  - refund
---

# Reversal workflow abuse

## Context

Refund APIs may not be idempotent; parallel refund and capture requests can double-credit when ledger updates are not atomic. Chargeback webhooks may be replayed if event ids are not stored.

## Tools

- **Burp Suite Turbo Intruder**: single-packet attack to fire concurrent refund and capture requests.
- **Burp Repeater**: tab groups to send grouped refund requests in parallel, and to replay chargeback callbacks.
- **curl**: parallel refund requests (backgrounded or driven with xargs).

## References

- PortSwigger Web Security Academy: Race conditions
- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic

