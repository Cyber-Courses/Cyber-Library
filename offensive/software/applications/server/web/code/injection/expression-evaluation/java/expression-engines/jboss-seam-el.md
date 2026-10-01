---
title: "JBoss Seam EL injection: remote command execution through Unified EL"
description: "Seam evaluates Unified EL expressions, and attacker-controlled EL reaching an interpolated value or a view parameter resolves arbitrary Java, giving command execution in the application server."
keywords:
  - JBoss Seam
  - Seam EL injection
  - Unified EL
  - expression language injection
  - remote command execution
---

# JBoss Seam EL

JBoss Seam builds on Unified Expression Language (`#{...}`), using it far more aggressively than a plain view layer: EL drives navigation, page actions, parameters, and interpolated messages across the framework. Seam also extended EL with parameterized method calls. The security consequence is that any place Seam evaluates an EL string built from untrusted input becomes a path to arbitrary Java, because EL resolution reaches object methods and, through them, the runtime. This is the mechanism behind the well-known Seam remote-command exposure, where an EL expression delivered through a request parameter is evaluated server-side.

## Vulnerable pattern

Seam evaluates EL in several sinks: `actionOutcome` and page-action values, interpolated status and faces messages, and application code that resolves an expression directly:

```java
// value derived from the request
Expression expr = expressions.createValueExpression(input);
Object result = expr.getValue();
```

When `input` carries attacker EL, Seam's resolver walks it against the full application context, including implicit objects Seam exposes.

## Reaching the runtime

Unified EL has no `import`, no `new`, and no class-literal syntax, so it cannot name `java.lang.Runtime` directly. The reachable path uses Seam's method-call support to go through reflection from a string literal: get `Class` via `forName`, look up the static `getRuntime` method, invoke it with a null receiver, then call `exec`:

```
#{''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')}
```

Reading output back wraps the returned `Process` stream through further reflective EL calls, constructing a reader or scanner over `getInputStream()` so the command result lands in whatever value Seam renders.

Historically this surface was reached through unauthenticated entry points such as the `actionOutcome` request parameter on Seam's default pages, where the parameter value is taken as an EL outcome and evaluated during navigation, so no application code has to opt in for the expression to run.

## Shell features

`Runtime.exec(String)` tokenizes on whitespace with no shell, so `$(...)`, pipes, and redirection do not expand, and the single-string form above runs one binary. Unified EL cannot write a `String[]` literal (no `new`, no array syntax), so shaping a `/bin/bash -c` invocation means building the argument array reflectively through `java.lang.reflect.Array` and passing it to the `exec(String[])` overload, which is far more verbose than the single-command form. In practice, redirect the single command's output to a file and read it back, or use the one-shot command where no shell features are required.

## References

- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
- [JBoss Seam reference documentation](https://docs.jboss.org/seam/latest/reference/html_single/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
