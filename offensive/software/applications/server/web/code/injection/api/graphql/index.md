---
title: "GraphQL abuse: resolver trust and query shape"
description: "A GraphQL endpoint executes a client-shaped query against per-field resolvers. The attack surface is where authorization lives, how resolver arguments reach backends, and how aliases and batching amplify a single request."
keywords:
  - GraphQL injection
  - resolver authorization
  - GraphQL arguments
  - alias abuse
  - field-level authorization
---

# GraphQL

GraphQL exposes one endpoint that runs an arbitrary, client-shaped query tree. The server walks the query field by field, calling a resolver for each, and the client decides which fields to request, in what nesting, under which aliases, and how many at once. That flexibility is the attack surface: authorization and validation that assume a fixed set of REST routes do not map cleanly onto a graph where the caller composes the operation.

## Where it goes wrong

The pages here follow the three places a GraphQL server misplaces its trust:

- **[Field authorization bypass](field-authorization-bypass.md)**: checks applied at the operation or HTTP edge but not on individual field and edge resolvers, so a caller reads or mutates objects outside their tenancy through the graph.
- **[Resolver argument forwarding abuse](resolver-argument-forwarding-abuse.md)**: a resolver passing its arguments straight into SQL, a command, a URL, or an internal API, a second-order injection hidden behind the schema's type checks.
- **[Alias and batch query abuse](alias-and-batch-query-abuse.md)**: aliases, batched operations, and nested duplicate fields used to amplify cost or brute-force values past limits that were only enforced per HTTP request.

## Why the graph changes the model

In REST, one route maps to one handler with one place to check access. In GraphQL, a single request can touch dozens of resolvers across many types, and each resolver is its own trust boundary. A schema that type-checks arguments proves they are well-formed, not that they are authorized or safe to forward. The recurring test is to run the same query as two different identities, and to trace each argument from the schema down to the backend it reaches.

## References

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
