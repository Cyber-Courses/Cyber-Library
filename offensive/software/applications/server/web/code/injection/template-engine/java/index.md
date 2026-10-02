---
title: "Java server-side template injection"
description: "SSTI in Java template engines: FreeMarker's Execute and ObjectConstructor built-ins and Velocity's reflection chain to Runtime.exec."
keywords:
  - Java SSTI
  - FreeMarker
  - Velocity
  - template injection RCE
---

# Java

Java template engines reach code execution through the platform's own classes rather than a scripting layer, so exploitation walks to `java.lang.Runtime` or `ProcessBuilder` and calls `exec`. Both engines here use `${ ... }` for interpolation, so `${7*7}` returning `49` is the shared probe (Velocity also responds to `#set($x=7*7)$x`).

The difference between them is the gadget: FreeMarker ships utility built-ins (`Execute`, `ObjectConstructor`, `new`) that construct and run objects directly, while Velocity has no such helper and is exploited by reflecting from an available class to `Runtime`.

## Engines

- **[FreeMarker](freemarker.md)**: the `Execute`, `new`, and `api` built-ins, and the `new_builtin_class_resolver` gate.
- **[Velocity](velocity.md)**: reflection from `$class.inspect(...)` to `Runtime.getRuntime().exec`.

## References

- Apache FreeMarker and Apache Velocity documentation
- PortSwigger Web Security Academy: Server-side template injection
