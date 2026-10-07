---
title: "Access control in web applications: endpoints, objects, properties, and broken authorization (BOLA/IDOR)"
order: 3
description: "Broken access control in server-side web code: endpoint, object, and property scope, plus trust toward headers and upstream systems."
keywords:
  - access control
  - authorization
  - BOLA
  - IDOR
  - broken access control
---

# Access control

**Access control** (authorization) answers whether a caller is allowed to perform an action on data. Failures are common and high impact. Organize testing and documentation by **where** the check fails:

| Subtopic | Question |
|----------|----------|
| [Endpoint level](endpoint-level/index.md) | Is this route or function restricted to the right roles or tenants? |
| [Object level](object-level/index.md) | May this user access **this specific record** (IDs, slugs, nested resources)? |
| [Property level](property-level/index.md) | Can the user change fields they should not (mass assignment, partial updates)? |
| [Trust boundary](trust-boundary/index.md) | Does the app believe headers, client certificates, or upstream identity without sound binding? |

