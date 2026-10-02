---
title: "Nested property injection: path-like keys that set deep fields on server objects"
description: Deep merge and dotted keys that set inner model fields the top-level allowlist does not name.
keywords:
  - deep merge
  - nested key
  - overposting
---

# Nested property injection

## Context

JSON merge for `PATCH` and ORM `assign` of nested dicts can write `a.b.c` where a flat allowlist only checked `a`. GraphQL `input` types and protobuf `struct` fields have the same merge semantics risk when the server merges user input into an existing object server-side.

## Theory

The dangerous state is: default nested object with sensitive defaults, and the merge overwrites a child key the client should not control. Some stacks use `lodash.merge` style deep merge; others replace whole sub-objects. The behavior is library-specific.

## Practice

### Target inner keys in a lab

- Build a `PATCH` body with a nested object path that mirrors the ORM or document shape (`user.profile.role`) and send it with a low-privilege token. Compare before/after reads of the object.

## Tools

- **curl**
- **Burp Suite**
