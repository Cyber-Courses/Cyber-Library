---
title: "gRPC abuse: reflection, metadata, and dynamic typing"
description: "gRPC dispatches protobuf messages to service methods. The attack surface is server reflection exposing the RPC map, metadata trusted for authorization, and dynamic message typing that decodes attacker-chosen types."
keywords:
  - gRPC injection
  - server reflection
  - gRPC metadata
  - protobuf Any
  - grpcurl
---

# gRPC

gRPC carries protobuf messages over HTTP/2 to strongly typed service methods. The binary framing and generated stubs make it feel closed, but the same properties that make it efficient, a discoverable service map, a metadata side channel, and dynamic message types, are where application code over-trusts the caller.

## The three surfaces

- **[Server reflection abuse](server-reflection-abuse.md)**: the reflection service handing out the full list of services, methods, and message descriptors, turning an opaque binary API into a browsable one with a tool like `grpcurl`.
- **[Metadata abuse](metadata-abuse.md)**: the per-call metadata map (`:authority`, custom headers, and, behind grpc-gateway/grpc-web, HTTP headers mapped in via the `Grpc-Metadata-` prefix) trusted for authorization, routing, or tenant selection even though the client sets it freely.
- **[Dynamic message confusion](dynamic-message-confusion.md)**: `google.protobuf.Any`, type registries, and `oneof` handling where the server decodes an attacker-chosen type into an unsafe handler, a confused deputy between message types.

## Why binary does not mean safe

The protobuf wire format and the generated client suggest a sealed contract, but nothing about binary framing enforces authorization or restricts which methods a caller may invoke. Reflection advertises the surface, metadata is attacker-controlled input that looks like infrastructure, and dynamic typing reintroduces the exact "decode untrusted bytes into a chosen type" problem that strong typing was meant to remove. The test for each is the same: assume the caller can name any method, set any metadata, and supply any type URL, then find where the server acts on that without checking.

## Tools

- **grpcurl**: listing services and invoking methods over gRPC.
- **grpcui**: interactive browser client for a gRPC server.
- **blackboxprotobuf (Burp extension)**: decoding and tampering protobuf messages in Burp.

## References

- [gRPC: Server Reflection](https://grpc.io/docs/guides/reflection/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
