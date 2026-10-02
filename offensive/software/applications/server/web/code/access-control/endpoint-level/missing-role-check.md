---
title: "Missing role and permission checks on HTTP handlers, controllers, and GraphQL resolvers"
description: Handlers that authenticate the caller but omit application-level permission checks for the specific route, mutation, or GraphQL field.
keywords:
  - missing authorization
  - function level authorization
  - vertical privilege escalation
  - role check
  - broken access control
---

# Missing role checks

## Context

The subject is already authenticated; the failure is that the code path runs a sensitive operation (admin report, state change, export) without matching the subject’s role, scope, or tenant to the operation. Sibling object-level BOLA is a different class: the route is “right” in name, but the object id is wrong. This page is for **function**-level gaps.

## Theory

Common shapes: a new route ships without a guard the older pattern used; a GraphQL field resolver omits a field-level check the type-level assumes; a microservice trusts the internal network and skips entry authorization; a versioned path duplicates behavior with a weaker policy matrix. Fuzz by role matrix (low vs high privilege accounts) and by HTTP method and content type on the same path.

## Practice

### Map route and role matrix in a lab

- With two test accounts in different roles, call the same path and method with the same body shape. If one response is `403` for the first account and `200` for the second on an admin-only function, the policy exists; if both return `200`, the function-level check is likely missing for that route set.

### Compare gateway and app enforcement

- When a gateway or BFF is in play, request the same internal route both through the gateway and (in a dev lab only) against a direct app port if exposed. A missing check often appears only on one path.

### GraphQL field surface

- List field names for a type in a test schema or doc bundle, then call each field with a low-privilege token. Field-level missing checks often show as `data` for fields that should be `errors` with an auth extension.

## Tools

- **curl**
- **Postman**
- **Burp Suite**
