---
title: "Nested resource access without ownership checks: IDOR across parent/child URLs"
description: Hierarchical paths where the parent is authorized but the child id is not verified to belong to that parent.
keywords:
  - nested resource
  - BOLA
  - object graph
---

# Nested object access

## Context

Paths like `/orgs/5/invoices/99` may check membership in org 5 but never join `invoices` to that org, so 99 from another org still returns. The failure is a missing invariant on the full key (org, child), not a single-table id only.

## Theory

ORM “include” of nested children and GraphQL resolvers that batch-load by child id only are common code shapes. The offensive test is to hold a valid child id from tenant A and place it under tenant B’s parent in the path.

## Practice

### Swap child id across parents in a lab

- Create child objects under org A. Note one child id. Use org B in the path with A’s child id. A `200` is a strong signal of a missing join check.

## Tools

- **Burp Suite**
- **curl**
