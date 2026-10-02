---
title: "JEXL injection: code execution through Apache Commons JEXL"
description: "Apache Commons JEXL expressions can resolve classes, construct objects, and invoke methods, so untrusted input into a JEXL evaluation reaches ProcessBuilder and the JVM runtime for command execution."
keywords:
  - JEXL
  - JEXL injection
  - Apache Commons JEXL
  - new() operator
  - JVM code execution
---

# JEXL

Apache Commons JEXL is a scripting and expression language for the JVM, used to let configuration, rules, or user-supplied formulas run against application objects. Unlike a math-only evaluator, JEXL is designed to interact with Java: expressions can read and write context variables, call methods on any object they can reach, index collections, and, through the `new` operator, construct arbitrary classes by name. That design is exactly what makes an injected JEXL expression a code-execution sink rather than a data-projection issue.

## Vulnerable pattern

```java
JexlEngine jexl = new JexlBuilder().create();
// expression text from the request
JexlExpression e = jexl.createExpression(input);
Object result = e.evaluate(context);
```

Anything the host places in the `JexlContext` is reachable, and because JEXL resolves class names, the attacker is not limited to the registered objects.

## Reaching the runtime

JEXL's `new` operator instantiates a class from its fully qualified name, and method calls chain off the result. A `ProcessBuilder` built and started this way runs a command in the application process:

```
new('java.lang.ProcessBuilder', ['id']).start()
```

An alternative pivots off any object's class to reach `Runtime` by reflection. The static `getRuntime` is not a method of the `Class` object that `forName` returns, so it is reached with `getMethod(...).invoke(null)` rather than chained directly:

```
''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')
```

To read command output back into the response, wrap the started process stream, for example constructing a `java.util.Scanner` over the process input stream and reading a delimited token, all expressible through the same `new`/method-call grammar.

## Shell features need an argument vector

`Runtime.exec(String)` and a single-string `ProcessBuilder` argument tokenize on whitespace and run with no shell, so `$(...)`, backticks, pipes, and redirection are passed as literal arguments and never expand. For anything needing a shell, build the argument list explicitly and let `/bin/bash -c` interpret it:

```
new('java.lang.ProcessBuilder', ['/bin/bash','-c','id > /tmp/o 2>&1']).start()
```

On Windows the equivalent vector is `['cmd.exe','/c','whoami > C:\\Windows\\Temp\\o.txt']`.

## Version behavior

The reachable grammar depends on the JEXL major version and how the host configured the engine. JEXL 2 evaluates this class-resolving, method-calling grammar by default. JEXL 3 introduced a more explicit permissions and sandbox model (`JexlPermissions`, restricted `JexlContext` and namespace resolvers), so whether `new` and arbitrary class resolution are reachable depends on whether the application left the permissive defaults or locked the engine down. Where a JEXL 3 engine is restricted, the reachable surface collapses to the methods and objects the host explicitly permitted, so enumerate what resolves (class construction, `getClass`, registered namespace functions) before committing to a full command payload. The template flavor (`JxltEngine`, `${...}` / `#{...}`) wraps the same expression grammar, so a `${...}` sink carries the same reach.

## Tools

- **Burp Suite**: Repeater to deliver JEXL new()/reflection payloads.
- **J2EEScan**: Burp extension with Java expression-injection checks.
- Manual JEXL payloads building ProcessBuilder or reflecting to Runtime.

## References

- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [Apache Commons JEXL: syntax reference](https://commons.apache.org/proper/commons-jexl/reference/syntax.html)
- [Apache Commons JEXL: permissions and sandboxing](https://commons.apache.org/proper/commons-jexl/apidocs/org/apache/commons/jexl3/introspection/JexlPermissions.html)
