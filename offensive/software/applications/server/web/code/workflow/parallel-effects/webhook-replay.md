---
title: "Webhook replay attacks: duplicate integration callbacks without nonces, clocks, or idempotent handlers"
description: Re-sending HTTP callbacks to integration endpoints that lack nonce, timestamp windows, or replay-resistant idempotency.
keywords:
  - webhook
  - replay attack
---

# Webhook replay

## Context

Partner systems POST “payment captured” or “user provisioned” events. If the receiver accepts the same JSON twice and the signature still verifies (or there is no signature), an attacker who captured one callback can replay it to grant duplicate entitlements.

## Theory

Defense in depth uses HMAC with timestamp, unique event IDs stored server-side, and idempotent handlers. Weakness is often at the **receiver**, not the sender.

## Practice

- In a lab, capture one valid callback with Burp; replay unchanged; then replay with a new `Idempotency-Key` header if the app honors it inconsistently.

## Tools

- **Burp Suite Repeater**
