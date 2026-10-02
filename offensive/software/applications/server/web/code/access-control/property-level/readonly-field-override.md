---
title: "Read-only field override: PATCH and merges that ignore immutability or server-owned fields"
description: OpenAPI or UI metadata marks a field read-only while the write handler still copies that key from the request body.
keywords:
  - read-only field
  - OpenAPI
  - overposting
---

# Read-only override

## Context

“Read only” in a generated client or in OpenAPI is not a server check. If the `UserUpdate` DTO in code still includes `createdAt` or `creditScore` in the bind set, a direct API call can set them.

## Theory

The same as mass assignment with a field that is *documented* as immutable. The offensive proof is a single `PATCH` with the supposedly read-only key.

## Practice

### Send documented read-only fields on the write route

- In a lab, add `id`, `createdAt`, or a `readOnlyFlag` from the `GET` response into a `PATCH` body and see if the server accepts the write and reflects it on `GET`.

## Tools

- **curl**
- **Burp Suite**
