---
title: "Document query JSON injection: parsing user JSON into database driver APIs"
description: APIs that pass req.body directly into ODM find() or aggregation without schema validation of keys and operators.
keywords:
  - NoSQL injection
  - JSON injection
---

# Document query JSON

## Context

**`Model.find(req.query)`** or **`JSON.parse`** into a filter object lets attackers supply **operator keys** and **pipeline** arrays. **Whitelist** top-level fields and **reject** keys starting with `$` where inappropriate.

## See also

- [NoSQL injection (parent)](index.md)
- [MongoDB operators](mongodb-operator-injection.md)
