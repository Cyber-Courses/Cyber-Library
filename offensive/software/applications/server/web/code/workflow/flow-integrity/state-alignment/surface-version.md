---
title: "Client workflow step vs server state machine version: skipping and jumping multi-step flows"
description: "Client-side step indicators that fall out of sync with a server-side workflow version or state machine."
keywords:
  - workflow
  - state machine
---

# Surface vs version

## Context

The SPA shows “Step 3 of 5” while the API’s workflow engine is on version 2 with different allowed transitions. Direct POSTs to `/step/4` may succeed if the server only checks session role, not currentState from the state table.

## Theory

Robust designs expose an opaque workflow token or server-generated step URL after each transition.

## Practice

- Complete steps 1–2 normally; then POST the step-4 payload while the UI still shows step 3 in a test tenant.

## Tools

- **Burp Suite**

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
