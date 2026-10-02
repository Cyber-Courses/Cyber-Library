---
title: "Workflow state alignment: reconciling UI, database rows, and session checkpoints in multi-step flows"
description: "Keeping UI, database records, and session checkpoints coherent across multi-step flows."
keywords:
  - workflow state
  - session
---

# State alignment

State alignment asks whether the **surface** (what the user sees), the **record** (authoritative row in the database), and the **session** (ephemeral checkpoint) all describe the same step of a workflow. Drift enables skipping, duplicate submission, or wrong-branch transitions.

## Pages

| Page | Focus |
|------|--------|
| [Surface vs version](surface-version.md) | Client step counter vs server workflow version |
| [Record status](record-status.md) | Row status fields vs side effects already executed |
| [Session checkpoint](session-checkpoint.md) | Session keys that gate steps without DB corroboration |

## Tools

- **Burp Suite (Repeater)**: replay steps while mutating the backing record to surface surface, record, and session drift.
- Manual testing with Burp Repeater and crafted payloads.

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic

