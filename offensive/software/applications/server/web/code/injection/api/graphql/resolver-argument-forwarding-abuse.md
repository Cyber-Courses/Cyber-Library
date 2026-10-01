---
title: "GraphQL resolver argument forwarding abuse"
description: "A resolver that passes its GraphQL arguments straight into SQL, a command, a URL, or an internal API turns a schema-validated query into ordinary injection, reached through the resolver and masked by the type system."
keywords:
  - resolver argument injection
  - GraphQL SQL injection
  - injection through resolver
  - resolver sink
  - argument forwarding
---

# Resolver argument forwarding abuse

The schema type-checks a resolver's arguments, which proves they are well-formed, not that they are safe to use. When a resolver forwards an argument into a backend, a SQL query, an OS command, an outbound URL, or an internal service call, without validating or parameterizing it, the graph becomes a delivery vehicle for the same injection classes found anywhere else, reached through a well-typed query that passes every schema check.

## Argument into a query

A `String` argument that the schema accepts flows into a concatenated query in the resolver:

```graphql
query {
  products(filter: "name LIKE '%a%'") { id name }
}
```

If the resolver builds SQL by concatenating `filter`, the argument carries a payload the schema never inspected, because to GraphQL it is simply a valid `String`. The mechanics of the resulting injection belong to the backend (see the [database](../../database/index.md) subtree); the point specific to GraphQL is that the type system gives false assurance, so arguments must be treated as untrusted at the resolver even when they are strongly typed.

## Argument into a request

Arguments that name a resource, a host, a path, or an identifier for a downstream call are the same hazard aimed at a different sink. A resolver that fetches a URL built from an argument is a server-side request forgery sink reached through the graph:

```graphql
query {
  linkPreview(url: "http://169.254.169.254/latest/meta-data/") { title }
}
```

and one that passes an argument into an internal service path reaches that service with the graph server's privileges.

## Why it hides

Forwarding abuse is easy to miss because the schema looks like validation. A reviewer sees typed arguments and assumes safety, while the resolver quietly interpolates them. Nested and relationship resolvers deepen the problem: an argument on a nested field may reach a different backend than the top-level query implies, so the sink is not where the query appears to point. The test is to trace each argument from the schema definition to the code that consumes it, and to send type-valid values carrying backend-specific metacharacters, watching for the downstream behavior rather than a schema error.

## References

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
