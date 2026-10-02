---
title: "Prisma filter injection"
description: "Spreading an untrusted object into a Prisma where argument lets a caller inject operators and relation filters, widening or bypassing the intended query without any raw SQL."
keywords:
  - Prisma where
  - filter injection
  - operator injection
  - relation filter
  - object spread
---

# Filter injection

Prisma's typed API is safe for values, but it trusts the shape of the objects it is given. A `where` argument is a nested object whose keys are fields and operators, and an endpoint that spreads an untrusted request body into it hands the caller control of the query's structure, not just a filter value.

## Spreading request input into where

```js
// UNSAFE: client controls the shape of the filter
const users = await prisma.user.findMany({
  where: { ...req.body.filter },
});
```

If the endpoint intended `{ email: "a@b.c" }`, a caller can send operators instead:

```json
{ "filter": { "email": { "not": "" } } }
{ "filter": { "role": { "equals": "ADMIN" } } }
{ "filter": { "OR": [ { "id": 1 }, { "id": 2 } ] } }
```

`{ "not": "" }` matches nearly every row, `OR` widens the set, and a key the endpoint never meant to be filterable (`role`, `isAdmin`) becomes a selector. Where `where` reaches across relations, a nested relation filter pivots the query to related records the caller should not be able to constrain on.

## Type coercion into a filter

Passing a value where Prisma expects a scalar can also change behavior: a field typed as a string that receives an object (`{"email": {"startsWith": "admin"}}`) is interpreted as an operator filter rather than an equality, so an endpoint doing `where: { email: req.query.email }` without validating the type lets the caller switch from equality to a prefix or negation match.

## Confirming the flaw

Send operator objects where the endpoint expects a scalar and compare the result set: a body of `{ "not": null }`, `{ "gt": 0 }`, or a nested `OR` that returns more rows than a plain value would proves the filter shape is attacker-controlled. The fix is to validate and pick specific fields and operators rather than spreading the request, so an endpoint that forwards the raw object is the pattern to look for.

## References

- [Prisma: Filtering and sorting](https://www.prisma.io/docs/orm/prisma-client/queries/filtering-and-sorting)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
