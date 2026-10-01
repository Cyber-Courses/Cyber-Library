---
title: "Apache Camel injection: code execution through route expression languages"
description: "Camel routes evaluate expressions in the Simple, OGNL, or SpEL languages, and untrusted input built into a route expression reaches bean and method invocation and, through them, the JVM runtime."
keywords:
  - Apache Camel
  - Camel Simple language
  - Camel OGNL
  - Camel SpEL
  - route expression injection
---

# Apache Camel

Apache Camel is an integration framework built around routes that move and transform messages. Routes use expressions throughout: predicates for content-based routing, dynamic endpoint URIs, header and body setters, and templated strings. Camel supports several expression languages for these, and which one is in play decides how far an injected expression reaches. The exposure is any route expression built from message content or configuration that an attacker can influence, and the ceiling depends on the language the route uses.

## The Simple language

Camel's default lightweight language is Simple (`${...}`). It began as a string templating and predicate language but supports bean and method invocation, which is where it becomes a code-execution surface rather than a formatting one. An injected Simple expression can call a method on a reachable bean or construct and invoke via the bean language:

```
${bean:myBean?method=process(payload)}
```

Where the message body or a header is interpolated into a Simple expression that the route then evaluates, the attacker controls the method call target and arguments among the beans the registry exposes.

## OGNL and SpEL components

When a route uses the OGNL or SpEL language components (`.ognl(...)`, `.spel(...)`, or `language:ognl` / `language:spel` endpoints), the injected expression is full OGNL or Spring Expression Language. Both resolve class names and invoke arbitrary methods, so they reach the runtime directly. OGNL:

```
(#rt=@java.lang.Runtime@getRuntime()).exec('id')
```

SpEL:

```
T(java.lang.Runtime).getRuntime().exec('id')
```

Reading output back wraps the returned `Process` stream in the same expression grammar (an `InputStreamReader`/`Scanner` over `getInputStream()`), landing the result in the message the route produces.

## Shell features need an argument vector

`Runtime.exec(String)` tokenizes on whitespace with no shell, so `$(...)`, pipes, and redirection do not expand. Build an explicit vector for shell behavior, shown in SpEL:

```
T(java.lang.Runtime).getRuntime().exec(new String[]{'/bin/bash','-c','id > /tmp/o 2>&1'})
```

and equivalently in OGNL with `new java.lang.String[]{...}`. On Windows use `{'cmd.exe','/c','whoami'}`. Because the reach is language-dependent, confirm which expression language the route evaluates before committing: a Simple-only sink limits you to the exposed beans and their methods, while an OGNL or SpEL sink gives direct class resolution and the full runtime.

## References

- [Apache Camel: Simple language](https://camel.apache.org/components/latest/languages/simple-language.html)
- [Apache Camel: SpEL language](https://camel.apache.org/components/latest/languages/spel-language.html)
- [Apache Camel: OGNL language](https://camel.apache.org/components/latest/languages/ognl-language.html)
