---
title: "JSP EL injection: JavaServer Pages expression language abuse"
description: "JavaServer Pages evaluates the unified EL in ${...} expressions, and when request data reaches an evaluated expression it exposes scoped objects for authorization bypass and reflection for code execution."
keywords:
  - JSP EL injection
  - JavaServer Pages EL
  - unified EL
  - "${} expression"
  - EL injection
---

# JSP EL

JavaServer Pages evaluates the unified expression language in `${...}`, resolving names against the page, request, session, and application scopes through the `pageContext`. The expression is evaluated immediately as the page renders. Injection appears when request data lands inside an expression the container then evaluates: a parameter reflected into a `${...}` context, a value passed to `application.evaluate` through the JSP `ExpressionFactory`, or a tag attribute built from user input. The container runs the resulting EL with the page's scopes in reach.

The reachable surface matches the servlet EL grammar. The implicit scope objects (`param`, `header`, `sessionScope`, `requestScope`, `applicationScope`, `pageContext`) expose the attributes an access decision reads, and from EL 2.2 onward method invocation reaches reflection and the host runtime.

- **[Authorization bypass](authorization-bypass.md)**: reaching the scope maps and `pageContext` to read and force the values a check depends on.
- **[Code execution](code-execution.md)**: using method invocation and reflection to reach a script engine or `ProcessBuilder`.

## Tools

- **Burp Suite**: Repeater and Intruder for injecting into ${...} JSP expressions.
- **J2EEScan**: Burp extension with Java EL injection checks.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
