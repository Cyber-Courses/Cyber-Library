---
title: "Object-level access control: IDOR, BOLA, and per-record authorization in APIs"
description: Insecure direct object references and related issues—unauthorized access to data identified by IDs, slugs, or relationships between records.
keywords:
  - object level authorization
  - BOLA
  - IDOR
  - insecure direct object reference
---

# Object-level access control

**Object-level** access control (BOLA / IDOR) is the failure to verify that the authenticated subject may access **this specific** resource. The server accepts a reference (ID, UUID, slug) but does not enforce ownership, tenancy, or role on **that** instance.

## Pages

- [Direct reference tampering (IDOR)](direct-reference-tampering.md)
- [Batch object operation](batch-object-operation.md)
- [Alternate key lookup](alternate-key-lookup.md)
- [Nested object without ownership check](nested-object-without-ownership-check.md)
- [Predictable encoded IDs](predictable-encoded-ids.md)
- [Public identifier guessing](public-identifier-guessing.md)
- [Relationship trust mischeck](relationship-trust-mischeck.md)

## See also

- [Access control (parent)](../index.md)
