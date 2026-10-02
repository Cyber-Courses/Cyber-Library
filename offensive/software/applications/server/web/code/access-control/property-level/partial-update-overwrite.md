---
title: "Partial update overwrite: JSON Merge Patch and PATCH clearing nested or sibling fields"
description: PATCH and JSON merge behavior that overwrites read-only, derived, or financial fields when only a subset of fields should change.
keywords:
  - PATCH
  - JSON merge patch
  - partial update
  - overposting
---

# Partial update overwrite

## Context

`PATCH` is not uniform across frameworks: JSON Merge Patch, custom shallow merge, and `null` to delete semantics differ. A client can intend to change `name` while the merge overwrites `balance` or `role` if those keys are accepted in the merge input object.

## Theory

The same mass-assignment class appears when the *operation* is “partial” but the object shape is the full model. `null` and absent key often mean different things; attackers probe both.

## Practice

### Send full document shape with one extra sensitive key

- In a lab, take a `GET` of the object, copy the JSON, add a `creditLimit` or `status` key that should be server-only, and `PATCH` it. Compare the next `GET`.

## Tools

- **curl**
- **Burp Suite**
