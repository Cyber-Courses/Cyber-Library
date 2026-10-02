---
title: "Workflow sequencing attacks: step skip, out-of-order access, approvals, omission, and reversals"
description: "Abuse of ordered workflows: skipping steps, calling steps out of order, bypassing approvals, omitting parameters, and renewal or reversal paths."
keywords:
  - step skip
  - workflow bypass
  - approval bypass
---

# Sequencing

**Sequencing** is the **timeline** axis of flow integrity: what must happen before what, which human or system approvals gate transitions, and how renewal or reversal paths interact with capture and settlement.

## Pages

| Page | Focus |
|------|--------|
| [Step skip](step-skip.md) | Forced browsing to later steps |
| [Out-of-order step access](out-of-order-step-access.md) | Revisit intermediate steps |
| [Parameter omission abuse](parameter-omission-abuse.md) | Optional fields that gate logic |
| [Approval bypass](approval-bypass.md) | Maker-checker and approvals |
| [Entitlement renewal bypass](entitlement-renewal-bypass.md) | Trials, renewals, re-verification gaps |
| [Reversal and refund abuse](reversal-and-refund-workflow-abuse.md) | Refunds, chargebacks, settlement races |

## Tools

- **Burp Suite (Repeater)**: replay and reorder captured steps to force sequencing bugs.
- Manual testing with Burp Repeater and crafted payloads.

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic

