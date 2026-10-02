---
title: "TOCTOU in web workflows: time-of-check to time-of-use races on balances, quotas, and approvals"
description: Time-of-check to time-of-use gaps when a workflow reads state, branches, then acts without holding a consistent lock.
keywords:
  - TOCTOU
  - race condition
---

# TOCTOU

## Context

The application loads a record (quota, balance, approval flag), validates it in memory, then writes an update in a second step. Another request changes the record between read and write. In assessments, demonstrate impact only on non-production data and with written scope for concurrency tests.

## Theory

Weak isolation shows up when validation and mutation are separate HTTP handlers or use different transactions without `SELECT … FOR UPDATE` or equivalent.

## Practice

- In a lab, issue two concurrent requests that both pass a “single use” check; observe duplicate effects in logs or staging DB.

## Tools

- **Burp Suite** (Turbo Intruder or parallel repeater tabs)
- **custom scripts** (httpx, parallel curl)
