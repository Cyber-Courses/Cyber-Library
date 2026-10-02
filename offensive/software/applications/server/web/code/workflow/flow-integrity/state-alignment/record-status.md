---
title: "Record status vs side effects: stale workflow flags, duplicate jobs, and inconsistent settlement in web apps"
description: "Database status columns that lag behind irreversible actions (email sent, inventory decremented, payment captured)."
keywords:
  - workflow
  - consistency
---

# Record status

## Context

A row stays `PENDING` while webhooks already shipped goods. Retrying “sync status” jobs may double-ship or re-invoice because the compensating action keys only on `status`, not on an effect ledger.

## Theory

Model idempotent side effects with unique business keys and append-only event logs, not a single mutable flag.

## Practice

- Trigger the “retry provisioning” admin button twice in staging while monitoring duplicate emails or duplicate license keys.

## Tools

- Application logs and staging DB queries

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
