---
title: "Alternate key lookup: IDOR via invoice numbers, emails, and secondary business keys"
description: Resolving users or resources by email, username, or slug in APIs that return or mutate data without a relationship check to the caller.
keywords:
  - user enumeration
  - alternate identifier
  - IDOR
  - BOLA
---

# Alternate key lookup

## Context

Lookup by `?email=` or `?handle=` is convenient and sometimes public. The BOLA shape is the same as id tampering: the response or mutation runs for a key the caller should not be able to target, and the same signal as account discovery may appear in error text.

## Theory

The server resolves a stable alternate key to an internal id and then continues without a policy join to the current subject. Batch “search” endpoints that return full objects for partial matches increase blast radius.

## Practice

### Query with a peer’s email in scope

- With two test accounts, call the lookup using the other user’s email or public handle. Compare status, body size, and timing to a non-existent key in the same request shape.

## Tools

- **curl**
- **Burp Suite**
