---
title: "GraphQL alias and batch query abuse"
description: "Aliases, batched operations, and nested duplicate fields let one request run the same work many times, amplifying cost and brute-forcing values past limits that were only enforced per HTTP request."
keywords:
  - GraphQL alias
  - batched queries
  - query cost
  - rate limit bypass
  - query depth
---

# Alias and batch query abuse

A GraphQL request can ask for the same field many times under different aliases, send many operations in one HTTP request, and nest fields deeply. Each repetition runs the resolver again. When a limit, a rate cap, a login-attempt counter, or a cost budget, is enforced per HTTP request rather than per resolver execution, these features collapse many logical operations into one request and slip under it.

## Aliasing to multiply one request

Aliases let the same field appear repeatedly in a single query, each with different arguments. A per-request rate limit sees one request; the server runs the resolver once per alias:

```graphql
mutation {
  a: login(user: "admin", password: "guess1") { token }
  b: login(user: "admin", password: "guess2") { token }
  c: login(user: "admin", password: "guess3") { token }
  # ... hundreds more aliases in one request
}
```

A counter that increments once per HTTP request never fires, so a credential-stuffing or one-time-code brute force that would be throttled over many requests runs in a handful. The same trick reads many objects at once by aliasing a by-ID field across a range of identifiers.

## Batching operations

Where the server accepts an array of operations in one request, the array is the amplifier instead of aliases, with the same effect on any per-request control:

```json
[
  {"query": "mutation { redeem(code: \"0001\") { ok } }"},
  {"query": "mutation { redeem(code: \"0002\") { ok } }"},
  {"query": "mutation { redeem(code: \"0003\") { ok } }"}
]
```

## Cost and depth amplification

Nesting and repetition also drive raw cost. A query that follows a cyclic relationship, or repeats an expensive field across many aliases, forces disproportionate database and CPU work from a small request, which is a denial-of-service lever where no query-cost analysis bounds execution:

```graphql
query {
  users {
    posts { author { posts { author { posts { id } } } } }
  }
}
```

## Confirming the amplification

Send a benign field under a growing number of aliases and watch response time and any limit counters: if the server processes all of them and the per-request control registers once, the amplification is real. For brute force, the proof is a throttled flow (login, coupon, one-time code) succeeding in far fewer HTTP requests than the limit should allow. Defenses that would stop this, per-resolver cost accounting, alias and depth caps, live in code and are often absent, which is why the per-request assumption is worth testing first.

## References

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10: Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
