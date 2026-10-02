---
title: "Workflow state alignment: reconciling UI, database rows, and session checkpoints in multi-step flows"
description: Keeping UI, database records, and session checkpoints coherent across multi-step flows.
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
