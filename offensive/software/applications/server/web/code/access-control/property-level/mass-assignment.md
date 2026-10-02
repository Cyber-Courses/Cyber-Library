---
title: "Mass assignment and overposting: binding attacker-supplied properties in APIs and forms"
description: Binding client JSON or form fields into persistence models without an allowlist, allowing role, owner, or price fields to change in one request.
keywords:
  - mass assignment
  - overposting
  - auto binding
  - privilege escalation
---

# Mass assignment

## Context

Frameworks map request bodies to ORM entities or document models. If the binder copies every key the client sends, a single `PATCH` can set `role`, `isAdmin`, `accountBalance`, or `ownerId` when those keys exist on the model. The signal is a writable sensitive field, not a wrong object id by itself (though you can chain both).

## Theory

Vulnerable code often uses one model or DTO for both low-privilege self-service updates and higher-trust fields on the same table. `PUT` replace and `PATCH` merge semantics differ: nested keys can land in deep graph properties a shallow key filter never names.

## Practice

### Add privilege keys next to valid fields in a lab

- Send a small JSON body that is normally allowed (for example, `displayName`) plus a second key the UI never shows, such as `role` or `isAdmin`, with a value that should be impossible for the account. Observe response code and a follow-up `GET` of the same object for field persistence.

## Tools

- **curl**
- **Burp Suite**
- **Postman**
