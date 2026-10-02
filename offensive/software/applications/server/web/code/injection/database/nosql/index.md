---
title: "NoSQL injection in web applications: operator objects, JavaScript evaluation, and unsafe filters"
description: Attacker-controlled structures passed to document databases and ODM query builders, MongoDB operators, `$where`, and aggregation pipelines.
keywords:
  - NoSQL injection
  - MongoDB
  - operator injection
---

# NoSQL injection

NoSQL injection arises when application code passes **objects** or **strings** from HTTP into a query API that interprets keys as **operators** (`$gt`, `$ne`, `$regex`) or evaluates JavaScript in `$where`. Typed APIs and allowlisted shapes kill most variants; **raw** `find(JSON.parse(body))`, merging client JSON into filters, and “flexible search” endpoints bring it back.

## Patterns

| Pattern | Example risk |
|---------|----------------|
| Operator injection | `{"username": {"$ne": null}, "password": {"$ne": null}}` |
| `$where` / server-side JS | Evaluation of user-influenced expressions where enabled |
| Aggregation stages | User-controlled pipeline arrays in reporting endpoints |

## Pages

| Page | Focus |
|------|--------|
| [MongoDB operators](mongodb-operator-injection.md) | `$ne`, `$regex`, auth bypass |
| [Document query JSON](document-query-json-injection.md) | Raw JSON into find/aggregate |
