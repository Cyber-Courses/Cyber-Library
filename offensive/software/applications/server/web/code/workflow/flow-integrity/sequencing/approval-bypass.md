---
title: "Approval bypass in workflows: self-approval, weak approver binding, stale tokens, and segregation-of-duties gaps"
order: 4
description: "Circumventing human or system approval steps: weak approver binding, self-approval, stale tokens, or missing segregation of duties in APIs."
keywords:
  - approval bypass
  - maker checker
  - workflow
---

# Approval bypass

## Context

Approval bypass targets transitions that require a second principal (manager sign-off, risk review). Typical failures include self-approval (same user as requester and approver), reused approval tokens, weak binding of approval identifiers to the request payload, and routes such as `POST /approve` that omit role or organization checks.

## Theory

Map states (for example pending → approved → settled) and which credential may fire each transition. Overlap with [Access control](../../../access-control/index.md) when the failure is a missing role on a route; this page emphasizes workflow graphs that include an explicit approver transition.

## Practice

### Self-approval attempt

- Create a request as user A, then call the approve endpoint authenticated as A when the product claims segregation of duties.

### Replay stale approval id

- Capture `approvalId` from a completed flow, start a new request, and substitute the old `approvalId` if the server accepts it without binding to a new payload hash.

## Tools

- **Burp Suite**
- **curl**

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
