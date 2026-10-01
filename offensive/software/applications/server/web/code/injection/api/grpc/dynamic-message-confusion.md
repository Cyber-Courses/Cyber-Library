---
title: "gRPC dynamic message confusion"
description: "When a server decodes google.protobuf.Any or resolves a oneof based on an attacker-chosen type, it can be steered to unpack bytes into an unintended handler, a confused deputy between message types."
keywords:
  - protobuf Any
  - type URL
  - oneof confusion
  - dynamic message
  - protobuf registry
---

# Dynamic message confusion

Protobuf is normally rigidly typed, but a few features reintroduce dynamic dispatch: `google.protobuf.Any` carries a type URL plus opaque bytes to be unpacked into whatever type the URL names, `oneof` lets one field hold one of several types, and registries resolve type names at runtime. When the server chooses how to decode a message from a value the caller controls, an attacker picks the type the bytes decode into, steering the request to a handler that was not meant to process it.

## Choosing the type with Any

An `Any` field defers the decision of what the payload is until unpack time, based on its `type_url`. If the server unpacks without restricting the allowed types, the caller selects one:

```json
{
  "payload": {
    "@type": "type.googleapis.com/internal.AdminCommand",
    "action": "promote",
    "target": "104"
  }
}
```

A handler that accepts a generic `Any` for a benign purpose, an event, an attachment, a metadata blob, and later unpacks it into a type chosen by the sender, can be made to materialize an internal command or privileged message type, which downstream code then acts on as if it were trusted.

## oneof and registry resolution

A `oneof` only holds the members its `.proto` declares, so selecting a different member is allowed contract behavior, not dynamic typing by itself. It becomes a flaw when the branches are not held to the same checks: if one branch carries an authorization or validation step that another skips, choosing the weaker branch reaches the action without the control the common path enforced. The genuinely dynamic case is registry-based resolution, where the server looks up a message or handler by a name taken from the request; supplying an unexpected name reaches a different implementation the contract never pinned. The `Any` case above and registry lookups select the type or handler from untrusted input; a `oneof` only matters where a branch is under-protected relative to its siblings.

## Why strong typing does not prevent it

The promise of protobuf is that a field has one type, so untrusted bytes cannot be reinterpreted. `Any`, `oneof`, and registries break that promise deliberately, to support extensibility, and in doing so they recreate the classic "decode attacker bytes as an attacker-chosen type" problem. The server must pin the set of types it will unpack and reject unknown `type_url` values; where it does not, the caller's type choice is the vulnerability. The test is to send a valid outer message whose `Any` or `oneof` selects an internal or unexpected type and observe whether the server unpacks and acts on it rather than rejecting the type.

## References

- [Protocol Buffers: Any](https://protobuf.dev/programming-guides/proto3/#any)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
