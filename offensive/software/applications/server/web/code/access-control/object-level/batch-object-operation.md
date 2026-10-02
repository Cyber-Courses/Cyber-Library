---
title: "Batch object operations: bulk endpoints that skip per-item authorization checks"
description: APIs that process many object ids in one request but apply authorization checks inconsistently across the batch.
keywords:
  - batch API
  - bulk operation
  - BOLA
  - IDOR
---

# Batch operations

## Context

Bulk `POST` or `DELETE` with an array of ids is efficient for clients. The server may authenticate the caller, validate the first id, and loop the rest with a weaker or missing check. One call can cross tenant or user boundaries.

## Theory

Offensive signals include mixed outcomes: some ids in the list succeed and some fail, a single error for the whole batch when per-item denial would be expected, or a 200 with a partial result object that hides which rows were touched.

## Practice

### Mix authorized and other-user ids in one body

- In a lab, build a body with two ids: one owned by the session and one from another test user. Send it to a bulk read or update endpoint and observe per-item error behavior versus full success on all items.

## Tools

- **curl**
- **Burp Suite**
