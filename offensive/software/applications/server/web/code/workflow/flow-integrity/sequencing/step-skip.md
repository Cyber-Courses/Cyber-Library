---
title: "Step skip in multi-step web flows: forced browsing to later endpoints without prerequisites"
description: Invoking a later workflow endpoint or state without completing prerequisite steps—onboarding, checkout, KYC, publishing gates.
keywords:
  - step skip
  - workflow bypass
  - forced browsing
---

# Step skip

## Context

**Step skip** is direct use of a **late** step’s HTTP endpoint or API operation **without** the **session** or **progress** artifacts the **server** thought it required (KYC before payout, shipping before charge, 2FA before bind). Testing is **authorized** only on programs you may attack.

## Theory

Servers often trust **client-visible** step order and store **weak** **progress** flags (`step=3` in a cookie, unsigned `currentStage` JSON). If the **state machine** is **not** **enforced** **server**-**side** **for** **every** **transition**, a **crafted** **POST** to `/api/order/finalize` succeeds **without** prior **calls**. **Idempotent** **retries** and **mobile** **clients** **skipping** **optional** **UI** **steps** **surface** the **same** **class**.

## Practice

### Number steps from traffic

- Capture a happy-path flow in Burp. Note each state-changing request. Replay **only** the **last** request with a **fresh** session; if it succeeds, the server did not bind prerequisites to that session.

### Direct navigation to deep links

- Open mid-flow URLs from bookmarks or `history` after completing the flow once; repeat with a new account that never completed early steps.

## Tools

- **Burp Suite**
- **browser**
