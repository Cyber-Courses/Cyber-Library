---
title: "JSF EL injection: JavaServer Faces expression language abuse"
description: "JavaServer Faces evaluates the unified EL in #{...} expressions, and when request data reaches an evaluated expression it exposes scoped objects for authorization bypass and reflection for code execution."
keywords:
  - JSF EL injection
  - JavaServer Faces EL
  - unified EL
  - "#{} expression"
  - EL injection
---

# JSF EL

JavaServer Faces (JSF) binds component attributes and values with the unified expression language, written `#{...}`. The value is a deferred expression: the framework evaluates it when the component renders or when a bound action fires, resolving names against the faces context and the scoped attribute maps. When an attacker influences the text that becomes a JSF expression, through a parameter echoed into a component attribute, a search filter rendered into a bound value, or a flow that compiles a user string with the application's `ExpressionFactory`, that evaluation runs attacker-chosen EL inside the request.

Two impacts follow from what the EL grammar reaches. The implicit scope objects (`sessionScope`, `applicationScope`, `requestScope`, `facesContext`) let an expression read and overwrite the attributes an access decision depends on. From EL 2.2 onward the grammar invokes methods, which reaches reflection and the host runtime.

- **[Authorization bypass](authorization-bypass.md)**: reaching the scoped attribute maps and the faces context to read and force the values a decision checks.
- **[Code execution](code-execution.md)**: using method invocation and reflection to reach a script engine or `ProcessBuilder`.

## Tools

- **Burp Suite**: Repeater and Intruder for injecting into #{...} JSF expressions.
- **J2EEScan**: Burp extension with Java EL injection checks.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
