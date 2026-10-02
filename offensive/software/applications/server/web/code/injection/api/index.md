---
title: "API injection: abusing structured request dispatch"
description: "Server code that parses and dispatches structured API requests decides what runs, for whom, and against which backend. When untrusted input shapes that dispatch, the contract itself becomes the attack surface, organized here by dispatch style."
keywords:
  - API injection
  - API authorization
  - RPC dispatch
  - GraphQL
  - SOAP
  - gRPC
  - JSON-RPC
---

# API

An API endpoint is a dispatcher. It takes a structured request, a graph query, an RPC envelope, a SOAP document, a protobuf message, decides which handler or resolver runs, binds the request's fields to that handler's arguments, and often calls further backends on the caller's behalf. Every one of those steps trusts the structure of the request, and when an attacker controls that structure, the dispatch logic itself is the vulnerability, separate from any injection in the data it carries.

The flaws here are rarely a single tainted string reaching a sink. They are mismatches: between the operation the caller named and the one the server runs, between where authorization is checked and where the sensitive work happens, between the shape the validator expected and the shape the dispatcher accepts. The result is reaching methods, fields, or objects the contract was supposed to keep out of reach.

## Organized by dispatch style

Each protocol parses and routes requests differently, so the abuse patterns group by style:

- **[GraphQL](graphql/index.md)**: a single endpoint executing a client-shaped query tree against per-field resolvers. The surface is field-level authorization, argument forwarding into backends, and the cost of alias and batch amplification.
- **[gRPC](grpc/index.md)**: protobuf messages dispatched to service methods. The surface is server reflection, trusted metadata, and dynamic message typing (`Any`, `oneof`).
- **[SOAP](soap/index.md)**: XML envelopes routed by action and body. The surface is action-versus-body dispatch confusion, header processing (WS-Security, WS-Addressing), and body parameter unmarshaling.
- **[JSON-RPC](json-rpc/index.md)**: a method name and params mapped to a server procedure. The surface is procedure-name resolution, parameter shape handling, and batch arrays.

## The recurring theme

Across all four, the strongest attacks exploit a boundary that the framework made invisible: authorization enforced at the HTTP edge but not at the resolver, a dispatch keyed on one field while execution reads another, a validator that saw a different structure than the one the handler finally consumed. Each page ties the abuse to the handler or resolver code and the validation boundary it slips past.

## Tools

- **Burp Suite**: intercepting and replaying structured API requests across protocols.
- **InQL**: GraphQL schema introspection and query generation in Burp.
- **grpcurl**: enumerating and calling gRPC services.
- **Postman**: composing SOAP, JSON-RPC, and REST style requests for dispatch testing.

## References

- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [OWASP Web Security Testing Guide: API Testing](https://owasp.org/www-project-web-security-testing-guide/)
