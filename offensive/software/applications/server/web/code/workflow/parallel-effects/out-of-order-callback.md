---
title: "Out-of-order webhooks and callbacks: causal ordering bugs in payment and provisioning integrations"
description: "Asynchronous events applied in arrival order when the business contract requires causal order (e.g. authorize before capture)."
keywords:
  - webhook
  - ordering
---

# Out-of-order callbacks

## Context

Two callbacks reference the same business key but arrive on different connections. If the handler applies “last write wins” without a monotonic state version, capture may be processed before authorize, or a refund before a charge settles, leaving accounts inconsistent.

## Theory

Compare with distributed-systems version vectors; many CRUD apps omit them on webhook handlers.

## Practice

- In staging, delay one callback artificially (slow proxy) and send the dependent event first; observe ledger or entitlement tables.

## Tools

- **Burp Suite** with **Throttling**
- **tc** / **netem** in a lab network