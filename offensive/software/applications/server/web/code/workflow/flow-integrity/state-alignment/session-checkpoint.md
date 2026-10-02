---
title: "Session-only workflow gates: step flags without authoritative database corroboration"
description: "Steps gated only by session flags that are not tied to the authoritative workflow row."
keywords:
  - session fixation
  - workflow
---

# Session checkpoints

## Context

`session['checkout_step']=3` unlocks the payment form, but the draft order row was deleted or rolled back. The user submits payment against an inconsistent cart, or the server accepts payment without re-reading inventory.

## Theory

Every mutating step should validate a server-stored token or order ID that ties session to row.

## Practice

- Open two independent sessions for the same account (two browsers or incognito windows, each with its own session cookie). In session A, advance to the gated step so its session holds `checkout_step=3` (or the equivalent flag). In session B, change the authoritative record out from under it: cancel the draft order, empty the cart, or move the row to a state that should revoke the step. Then submit the mutating request from session A, whose stale checkpoint still says the step is unlocked. If the server honors the session flag instead of re-reading the row, the inconsistent action commits.

## Tools

- Two browsers or incognito windows (distinct sessions) against the same account in staging

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
