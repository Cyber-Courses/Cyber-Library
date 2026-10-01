---
title: "GraphQL field authorization bypass"
description: "When authorization is enforced at the operation or HTTP edge instead of on individual field and edge resolvers, a caller reads or mutates objects outside their tenancy by navigating the graph to them."
keywords:
  - GraphQL BOLA
  - field-level authorization
  - resolver authorization
  - IDOR GraphQL
  - broken object level authorization
---

# Field authorization bypass

GraphQL resolves a query field by field, and each field resolver is its own trust boundary. When access control is applied once, at the HTTP layer or on the top-level operation, but not on the resolvers that fetch related objects, a caller reaches data outside their tenancy by walking the graph to it. This is broken object-level authorization (BOLA/IDOR) in graph form, and it is the most common and highest-impact GraphQL flaw.

## Navigating to unauthorized objects

A query is authorized to fetch the caller's own object, but a nested field exposes a neighbor. If the resolver for that nested field fetches by ID without re-checking ownership, the caller reads it:

```graphql
query {
  me {
    id
    organization {
      members { id email role }   # are these resolvers checking the caller's membership?
    }
  }
}
```

The direct form supplies an identifier the caller should not own. Where object IDs are global and guessable or enumerable (sequential, or exposed elsewhere in the graph), requesting another tenant's node tests whether the resolver authorizes by identity or merely by the shape of the query:

```graphql
query {
  node(id: "VXNlcjoxMDQ=") {       # another tenant's user, base64 of User:104
    ... on User { email role }
  }
}
```

## Mutations are the same gap, with side effects

Field-level gaps on mutations are worse, because the missing check now writes. A mutation authorized to update the caller's profile may accept an `id` or a nested input that targets another object:

```graphql
mutation {
  updateUser(id: "104", input: { role: ADMIN }) { id role }
}
```

If the resolver loads the target by `id` and applies the change without confirming the caller may edit that user, the request escalates privileges or tampers with another tenant's data.

## Confirming the flaw

The reliable test uses two identities. Capture a query that returns one user's object graph, then replay it as a second, unrelated user with the first user's IDs substituted. A response that returns the first user's fields, rather than an authorization error or null, proves the resolver trusts the query shape over the caller's identity. Introspection, where it is enabled, maps the fields and edges worth probing; where it is disabled, the schema is still recoverable from error messages and client bundles. Each resolver that fetches by a caller-supplied ID or navigates an edge is a candidate, so enumerate them rather than testing only the top-level operation.

## References

- [OWASP API Security Top 10: Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
