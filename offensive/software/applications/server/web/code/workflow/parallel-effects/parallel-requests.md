---
title: "Parallel HTTP requests against workflow state: racing checkout, transfers, and step-gated operations"
description: Deliberate fan-out of HTTP requests to trigger races on session, cart, or step-gated operations.
keywords:
  - race condition
  - parallel requests
---

# Parallel requests

## Context

Attackers send many simultaneous requests against the same session or resource identifier to win races before server-side locks or queues serialize work. Common targets include checkout, transfer, role promotion, and ticket issuance.

## Theory

Effectiveness depends on connection pool size, app server threading, and whether the datastore enforces uniqueness constraints.

## Practice

- In scope, script 20–50 parallel POSTs to a staging endpoint and compare row counts and audit logs to single-threaded behavior.

## Tools

- **Burp Suite** Turbo Intruder
- **Python asyncio** or **GNU parallel** with curl