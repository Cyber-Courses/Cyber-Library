---
title: "gRPC server reflection abuse"
order: 1
description: "When a gRPC server exposes the reflection service in an environment where it should not, an attacker enumerates every service, method, and message descriptor, turning an opaque binary API into a browsable one."
keywords:
  - gRPC reflection
  - grpcurl
  - ServerReflection
  - API discovery
  - protobuf descriptors
---

# Server reflection abuse

gRPC server reflection is a built-in service that answers, at runtime, with the server's full set of services, methods, and message descriptors. It exists so tools can call an API without the `.proto` files on hand. When it is left enabled on an exposed endpoint, it hands the same map to an attacker, removing the one obstacle that binary framing otherwise puts in front of discovery.

## Enumerating the surface

With reflection on, a single tool call lists every service and method, then the message types for each. `grpcurl` drives it directly:

```
grpcurl -plaintext target:50051 list
grpcurl -plaintext target:50051 list admin.AdminService
grpcurl -plaintext target:50051 describe admin.AdminService.DeleteTenant
```

`list` returns the services, `list <service>` the methods, and `describe` the request and response message shapes. From nothing but a reachable port, this produces the complete RPC surface, including internal or administrative services that were never meant to be called by outside clients and that have no other public documentation.

## Why it matters beyond convenience

Binary gRPC without the `.proto` is genuinely hard to explore: a caller must guess method names and message fields byte for byte. Reflection removes that cost entirely, so a service whose only protection was obscurity, an admin method on the same server, a debug RPC, an internal management interface, becomes trivially callable once its name and message shape are known. Discovery is not the compromise by itself, but it is the step that makes every other gRPC attack (calling a method the caller should not, crafting a valid message for it) practical. After enumerating, invoke a discovered method with a crafted request:

```
grpcurl -plaintext -d '{"tenant_id":"104"}' target:50051 admin.AdminService.DeleteTenant
```

## Confirming exposure

Point `grpcurl ... list` at the endpoint. A populated service list confirms reflection is enabled and reachable; an `Unimplemented` error for the reflection service means it is off, and the surface must instead be recovered from client bundles, captured traffic, or leaked `.proto` files. Reflection is frequently enabled in development configurations and left on in production, so it is worth checking before assuming a gRPC endpoint is opaque.

## Tools

- [grpcurl](https://github.com/fullstorydev/grpcurl)
- [grpcui](https://github.com/fullstorydev/grpcui)

## References

- [gRPC: Server Reflection](https://grpc.io/docs/guides/reflection/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
