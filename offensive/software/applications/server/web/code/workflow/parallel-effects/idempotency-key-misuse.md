---
title: "Idempotency-Key misuse in APIs: wrong scoping, ignored keys on mutations, and tenant collisions"
description: "APIs that accept Idempotency-Key headers but scope them wrong, ignore them on mutating verbs, or collide keys across tenants."
keywords:
  - idempotency
  - API
---

# Idempotency keys

## Context

Clients send `Idempotency-Key` (or similar) so retries do not double-charge. Bugs appear when keys are ignored on PATCH, shared across users, truncated, or cleared after a short TTL while the client still retries.

## Theory

Correct behavior: same key + same body → same effect; same key + different body → 409 or clear error. Many implementations skip the body hash.

## Practice

- Replay the same POST with the same key and a changed amount in a sandbox; compare responses and backend state.

## Tools

- **curl** with custom headers
- **Burp Suite**

## References

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic