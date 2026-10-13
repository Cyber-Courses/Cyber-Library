---
title: "gRPC metadata abuse"
order: 2
description: "When a gRPC server trusts per-call metadata such as the authority or custom headers for authorization, routing, or tenant selection, a client that sets those values freely spoofs identity or reaches internal endpoints."
keywords:
  - gRPC metadata
  - grpc-metadata
  - authority header
  - tenant spoofing
  - metadata authorization
---

# Metadata abuse

Every gRPC call carries a metadata map: key-value pairs sent alongside the message, including reserved entries like `:authority` and arbitrary custom keys. Metadata is a convenient side channel for cross-cutting concerns, so application code often reads it for authorization, tenant selection, or routing. The catch is that the client controls it completely, so any decision made from unsigned metadata trusts attacker input that merely looks like infrastructure.

## Trusted claims in metadata

A common pattern puts identity or role in a custom metadata key set by a gateway, then reads it in a downstream interceptor. If that service is reachable directly, or if the gateway does not strip client-supplied copies, the client sets the key itself:

```
grpcurl -plaintext \
  -H 'x-user-id: 1' -H 'x-user-role: admin' \
  -d '{}' target:50051 billing.BillingService.ListAllInvoices
```

The server reads `x-user-role` and authorizes as admin, because it treated an internal convention as if clients could not forge it. The same applies to tenant keys (`x-tenant-id`), where overriding the value reads or writes another tenant's data.

## Authority and routing

The `:authority` pseudo-header and host-like metadata are sometimes used to pick a backend, a virtual service, or a tenant shard. Setting it to an internal name can route a call to a service the client should not reach, or select a privileged tenant context:

```
grpcurl -authority 'internal-admin.svc.cluster.local' -plaintext \
  -d '{}' target:50051 admin.AdminService.Health
```

## Why it is trusted when it should not be

Metadata feels like transport plumbing, so developers treat it as trustworthy the way they might treat a server-set environment variable. But on the wire it is just client input with a different name. A gateway that injects verified claims must also strip any client-supplied copies of the same keys, and downstream services must distinguish metadata they signed from metadata a caller provided. The test is to add or overwrite the authorization, tenant, and authority metadata on a direct call and see whether the server acts on it; if an interceptor reads a key without verifying its origin, the call is authorized on forged input.

## Tools

- **grpcurl**: setting custom metadata and the authority with -H and -authority on a direct call.
- **grpcui**: adding or overriding metadata through an interactive client.
- **Burp Repeater**: tampering mapped Grpc-Metadata- headers behind grpc-gateway or grpc-web.

## References

- [gRPC: Metadata](https://grpc.io/docs/guides/metadata/)
- [OWASP API Security Top 10: Broken Authentication](https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/)
