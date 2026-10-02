---
title: "Property-level access control: mass assignment, overposting, and writable sensitive fields"
description: Unauthorized modification of object fields through APIs, mass assignment, overposting, and filter bypasses on which columns may change.
keywords:
  - property level authorization
  - mass assignment
  - overposting
  - BOLA property
---

# Property-level access control

**Property-level** issues occur when a user may change **fields** they should not: `role`, `ownerId`, `price`, `isAdmin`, or internal status flags. The route may be allowed for the user, and the object ID may even be theirs, but the update must still **restrict** which properties are legal for that role and operation.

## Pages

- [Mass assignment](mass-assignment.md)
- [Allowlist and denylist bypass](allowlist-denylist-bypass.md)
- [Nested property injection](nested-property-injection.md)
- [Partial update overwrite](partial-update-overwrite.md)
- [Property pollution](property-pollution.md)
- [Property type confusion](property-type-confusion.md)
- [Readonly field override](readonly-field-override.md)
- [Writable role, owner, and billing fields](writable-sensitive-fields.md)
