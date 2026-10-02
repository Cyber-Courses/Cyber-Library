---
title: "Session-only workflow gates: step flags without authoritative database corroboration"
description: Steps gated only by session flags that are not tied to the authoritative workflow row.
keywords:
  - session fixation
  - workflow
---

# Session checkpoints

## Context

`session['checkout_step']=3` unlocks the payment form, but the draft order row was deleted or rolled back. The user submits payment against an inconsistent cart, or the server accepts payment without re-reading inventory.

## Theory

Every mutating step should validate a **server-stored** token or order ID that ties session to row.

## Practice

- Clear server-side session store for one tab while another tab advances the UI; submit from the stale tab in a lab.

## Tools

- Two browsers or incognito windows against the same account in staging
