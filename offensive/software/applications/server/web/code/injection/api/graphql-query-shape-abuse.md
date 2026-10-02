---
title: "GraphQL query shape abuse: introspection, deep nesting, alias fan-out, and batching"
description: Exploiting client-controlled query shape in GraphQL—schema introspection for recon, deeply nested cyclic selections, alias multiplication, and batched operations that turn one HTTP request into thousands of resolver and database calls.
keywords:
  - GraphQL
  - introspection
  - denial of service
  - query depth
  - alias batching
  - resolver amplification
---

# GraphQL query shape

GraphQL inverts who decides the response. The server publishes a schema; the **client** sends a selection set describing exactly which fields, how deeply nested, and how many times it wants them resolved. Every field in that set is backed by a **resolver**—often a function that hits a database, a cache, or another service. Because the cost of a query is set by its shape and the shape is attacker-controlled, a single well-formed request can be made arbitrarily expensive, and the schema that makes this possible can usually be read back in full.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Issuing resource-exhausting queries against systems without written authorization is unlawful.

## Overview

Two properties combine into the attack surface:

1. **The client picks the field tree.** There is no fixed handler whose cost the server controls; the resolver graph runs whatever the selection set asks for.
2. **The schema is self-describing.** Unless disabled, introspection returns every type, field, argument, and relationship—the exact map needed to build an expensive or sensitive query.

The result is that reconnaissance and amplification feed each other: introspection reveals the cyclic relationships and list fields, and those are precisely what a denial-of-service or fan-out query exploits.

## Reconnaissance with introspection

The first step against any GraphQL endpoint is to pull the schema. The `__schema` meta-field enumerates everything:

```graphql
query {
  __schema {
    types { name kind fields { name args { name type { name } } } }
    queryType { name }
    mutationType { name }
  }
}
```

A compact probe for the type list alone:

```graphql
{ __schema { types { name } } }
```

When full introspection is disabled, partial field discovery still leaks structure through error messages. GraphQL servers commonly return **"did you mean"** suggestions on a mistyped field, so fuzzing field names against a type reconstructs the schema piecemeal:

```graphql
{ user { emai } }      # error: Cannot query field "emai"... did you mean "email"?
```

Introspection reveals the three things that matter for amplification: **list-returning fields** (each multiplies work), **object relationships that form cycles** (`user → posts → author → posts …`), and **mutations** exposed alongside queries.

## Deeply nested query DoS

When two types reference each other, the schema contains a cycle, and a selection set can walk it as deeply as the parser allows. Each level multiplies the resolver count:

```graphql
query {
  user(id: "1") {
    posts {
      author {
        posts {
          author {
            posts {
              author { posts { title } }
            }
          }
        }
      }
    }
  }
}
```

If every `posts` resolver issues its own database query, a tree a dozen levels deep expands into an exponential number of round-trips from a single request. Where the server enforces a **depth limit** but no **complexity/cost limit**, you stay just under the depth cap and widen instead—selecting many expensive fields and lists at each permitted level so total work still blows up.

## Alias fan-out

Aliases let the same field be requested many times in one operation under different response keys. Because each alias is resolved independently, aliasing multiplies the work of an expensive field without adding depth:

```graphql
query {
  a1: expensiveReport(range: "all") { total }
  a2: expensiveReport(range: "all") { total }
  a3: expensiveReport(range: "all") { total }
  # ... repeated hundreds or thousands of times
}
```

Alias fan-out also defeats naive per-field rate limiting and is a classic **credential-stuffing / brute-force amplifier**: a single HTTP request can carry hundreds of `login` or `redeemCoupon` mutations under distinct aliases, each a separate attempt, bypassing controls that count requests rather than operations:

```graphql
mutation {
  t1: login(user: "admin", pass: "Password1") { token }
  t2: login(user: "admin", pass: "Password2") { token }
  t3: login(user: "admin", pass: "Password3") { token }
}
```

Where the server caps alias count per request, splitting across batched operations (below) restores the volume.

## Batching abuse

Many GraphQL servers accept a **JSON array** of operations in one HTTP request and execute them all:

```json
[
  {"query": "mutation { login(user:\"admin\", pass:\"a\"){token} }"},
  {"query": "mutation { login(user:\"admin\", pass:\"b\"){token} }"},
  {"query": "mutation { login(user:\"admin\", pass:\"c\"){token} }"}
]
```

When depth, complexity, and alias limits are enforced **per operation** but batching is unbounded, the per-operation ceilings are irrelevant—send a thousand modest operations in one request. Batching stacks with aliasing: each batched operation carries its own alias fan-out, and the product of the two is the real amplification factor. For throttling that keys on HTTP requests, both aliasing and batching collapse an attack that should take thousands of requests into one.

## Measuring the effect

Query-shape abuse is confirmed the same way as any resource attack—by differential timing and error behavior:

- **Baseline vs. payload timing.** Send a shallow query, then the nested/aliased variant, and compare response time; a steep nonlinear climb as you add levels or aliases confirms per-level resolver multiplication.
- **Partial failures.** Batches that return some results and time out on others reveal the cost ceiling and where it sits (per operation vs. per request).
- **Introspection diffing.** Re-running introspection against staging vs. production shows whether the endpoint was left self-describing where it should not be.

Keep payloads scaled to what the authorization explicitly permits; the goal on an engagement is to demonstrate the multiplier, not to take the service down.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** with the **[InQL](https://github.com/doyensec/inql)** extension for introspection parsing, query generation, and batching/alias payloads.
- **[GraphQL Voyager](https://github.com/graphql-kit/graphql-voyager)** to visualize the type graph and spot cycles for nesting attacks.
- **[graphql-cop](https://github.com/dolevf/graphql-cop)** and **[clairvoyance](https://github.com/nikitastupin/clairvoyance)** to audit endpoints and reconstruct schemas when introspection is disabled.
- **[Altair](https://altairgraphql.dev/)** / **[GraphiQL](https://github.com/graphql/graphiql)** for interactive schema exploration and crafting selection sets.

## References

- [GraphQL Specification](https://spec.graphql.org/)
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [PortSwigger Web Security Academy: GraphQL API vulnerabilities](https://portswigger.net/web-security/graphql)
- [CWE-770: Allocation of Resources Without Limits or Throttling](https://cwe.mitre.org/data/definitions/770.html)
