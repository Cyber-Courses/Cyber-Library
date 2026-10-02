---
title: "API dispatch abuse: GraphQL query shape, schema authorization gaps, and gRPC reflection"
description: Structured API entry points—GraphQL resolver graphs and gRPC/protobuf dispatch—where the query shape, the schema, and the request envelope steer which resolvers and downstream calls run, with which objects, and at what cost.
keywords:
  - GraphQL
  - gRPC
  - API abuse
  - introspection
  - BOLA
  - server reflection
---

# API dispatch

**Structured APIs** replace hand-rolled HTTP routing with a schema and a dispatch layer. A GraphQL server exposes one endpoint and lets the *client* describe the shape of the response; a gRPC server exposes typed methods and unmarshals protobuf into handlers. In both cases the attacker's leverage is not a string spliced into a backend command but the **envelope itself**—the query tree, the selected fields, the method name, the metadata headers—which the server trusts to decide what work to do and whose data to touch.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Probing APIs you are not permitted to test is unlawful.

## Overview

A schema-driven API moves decisions that a REST service hard-codes into server-controlled code out to the request:

- **Which fields are returned** is chosen by the client's selection set, not the endpoint.
- **How much work runs** depends on the depth, aliasing, and batching of the query, not a fixed handler.
- **Which methods exist** can be enumerated at runtime from the schema (GraphQL introspection, gRPC server reflection) rather than guessed.
- **Who the caller is** often rides in a metadata header or a resolver-local check rather than a single gate at the edge.

Each of those shifts creates an offensive surface that the three pages below cover. The decisive question when assessing a structured API is always: *what does the client control about dispatch, and where—if anywhere—is the authorization and cost decision actually enforced?*

This hub is for structured API entry: GraphQL query shape and resolver graphs, gRPC and protobuf unmarshaling, and similar schema-driven envelopes. Raw HTTP line parsing is **[HTTP](../http/index.md)**; string-built SQL is **[Database](../database/index.md)**; object-level authorization failures on resolvers, when the root cause is a missing ownership check rather than injection, cross into **[Access control](../../access-control/index.md)**.

## Why it reaches sensitive work

- **The schema is a map.** Introspection and reflection hand the attacker the full type system, every query, mutation, and method—turning blind enumeration into a lookup.
- **Cost is client-chosen.** Nesting, aliases, and batched operations let one HTTP request fan out into thousands of resolver invocations or database round-trips.
- **Authorization is diffuse.** A single "is logged in" middleware at the edge says nothing about whether the *specific object* a nested resolver loads belongs to the caller.
- **Envelopes are trusted.** gRPC metadata and GraphQL context are frequently read as authoritative identity or tenant routing, the same trust-boundary mistake made with HTTP headers.

## Impact

Depending on the sink, the payoff ranges from **resource exhaustion** (a deeply nested or alias-multiplied query that saturates CPU or the database) to **broken object-level authorization** (reading or mutating other tenants' records through under-guarded resolvers) to **reconnaissance** (a production schema or service list that should never have been reachable). The ceiling is set by what the resolver graph can reach: internal services behind a gateway, cross-tenant rows in a shared datastore, and administrative mutations exposed in the same schema as read queries.

## Pages

| Page | Focus |
|------|--------|
| [GraphQL query shape](graphql-query-shape-abuse.md) | Introspection, deep nesting, alias fan-out, and batched operations that multiply resolver and database work into denial of service |
| [GraphQL authorization](graphql-authorization-gaps.md) | Field- and resolver-level BOLA: edge authentication that never scopes the object a nested resolver loads, and dataloader batching that hides it |
| [gRPC reflection and metadata](grpc-reflection-and-metadata-abuse.md) | Server reflection enumeration, calling undocumented methods, and metadata/auth headers trusted as security decisions |

## References

- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [GraphQL Specification](https://spec.graphql.org/)
- [gRPC Server Reflection Protocol](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
- [CWE-770: Allocation of Resources Without Limits or Throttling](https://cwe.mitre.org/data/definitions/770.html)
