---
title: "Writable role, owner, and billing fields: privilege escalation via updatable API properties"
description: A focused mass-assignment family where role, ownership, or money-related columns change through generic update APIs.
keywords:
  - role escalation
  - owner id tampering
  - payment tampering
  - mass assignment
---

# Writable sensitive fields

## Context

`role`, `ownerId` / `userId`, and billing fields (`amount`, `price`, `plan`, `discount`) are high-impact when writable by a user-facing update route. The pattern is still overposting; the page name highlights business impact and audit priority in triage.

## Theory

E-commerce and subscription code paths often use one “update order” or “update subscription” handler with a large bound object. Financial fields and ownership are sometimes optional keys in the same DTO the client uses for harmless edits.

## Practice

### Single-field probes on a cart or subscription object in a lab

- `PATCH` with only `{"total":1}` or `{"ownerId":<other_user_id>}` against an object the session should only partially control. Observe `200` and a follow-up read. Chain with BOLA on `id` when the route does not fix the object either.

## Tools

- **Burp Suite**
- **curl**
