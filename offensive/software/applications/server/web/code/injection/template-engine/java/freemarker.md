---
title: "FreeMarker server-side template injection"
description: "Exploiting FreeMarker SSTI: confirming with ${7*7}, command execution via the Execute, new, and api built-ins, and bypassing the new_builtin_class_resolver restriction."
keywords:
  - FreeMarker SSTI
  - Execute built-in
  - new built-in
  - freemarker.template.utility.Execute
  - new_builtin_class_resolver
---

# FreeMarker

FreeMarker is a common Java engine (Spring, Struts, standalone). Confirm with `${7*7}` returning `49`. FreeMarker's `?` built-ins include several that construct and invoke Java objects, which is the route to RCE.

The classic primitive is the `Execute` utility, instantiated with the `new` built-in and then called:

```freemarker
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id") }
```

`"freemarker.template.utility.Execute"?new()` builds an instance of a class that runs a command when called, and `ex("id")` executes it. Two other built-ins give alternative paths. `?api` exposes the underlying Java API of a value, allowing access to a classloader:

```freemarker
${ "freemarker.template.utility.ObjectConstructor"?new()("java.lang.ProcessBuilder","id").start() }
```

`ObjectConstructor` constructs an arbitrary object from a class name and constructor arguments, so building a `ProcessBuilder` and calling `.start()` runs the command. A third form reaches a `ProcessBuilder` through `?api.getClass()` and reflection when the utility classes are blocked.

FreeMarker added `new_builtin_class_resolver` to restrict which classes `?new` can instantiate. When the application sets it to `TemplateClassResolver.SAFER_RESOLVER` or `ALLOWS_NOTHING_RESOLVER`, `Execute` and `ObjectConstructor` are rejected and `?new` is effectively closed. The `?api` built-in is likewise gated by the `api_builtin_enabled` setting, off by default. On a hardened configuration, test each built-in: if all are blocked, the injection is limited to reading exposed data model variables. On a default or permissive configuration the `Execute?new()` one-liner is immediate RCE.

## Tools

- tplmap, SSTImap

## References

- Apache FreeMarker documentation: ?new, ?api, new_builtin_class_resolver
- PortSwigger Web Security Academy: Server-side template injection
