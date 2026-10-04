---
title: "Java web framework expression language injection: SpEL, OGNL, MVEL, and unified EL"
description: "Java web frameworks evaluate expression language drawn from request data, from the unified EL of JSP and JSF to the framework engines SpEL, OGNL, and MVEL that reach the host runtime."
keywords:
  - expression language injection
  - Java web framework EL
  - SpEL injection
  - OGNL injection
  - unified EL
---

# Web frameworks

Java web frameworks lean on expression languages to move data between the request, the controller, and the view. A page tag resolves a property path, a controller maps a request parameter onto an object graph, an annotation names a rule to evaluate. Each of those is an expression engine, and when the string it evaluates comes from request data the grammar becomes an execution surface inside the application process.

Two families appear here. The unified EL defined by the servlet stack and shared by JSP and JSF (`${...}` in JSP, `#{...}` in JSF) was built to read and write scoped attributes, and from EL 2.2 onward it invokes methods, which reaches reflection and the host runtime. The framework engines (SpEL in Spring, OGNL in Struts2 and older WebWork, MVEL in several binding and rule layers) are full object languages: they name arbitrary classes, construct objects, and call methods, so the baseline reachable surface already includes `java.lang.Runtime` and `ProcessBuilder`.

The route from an injected expression to impact is engine-specific, so the subtrees are split by engine:

- **[MVEL](mvel.md)**: a Java-like expression language with direct access to Java types, evaluated through `MVEL.eval` or a compiled expression.
- **[JSF EL](jsf-el/index.md)**: the `#{...}` unified EL of JavaServer Faces, reaching scoped objects for [authorization bypass](jsf-el/authorization-bypass.md) and reflection for [code execution](jsf-el/code-execution.md).
- **[JSP EL](jsp-el/index.md)**: the `${...}` unified EL of JavaServer Pages, covering [authorization bypass](jsp-el/authorization-bypass.md) and [code execution](jsp-el/code-execution.md).
- **[OGNL](ognl/index.md)**: the Object-Graph Navigation Language of Struts2, covering a [directory listing](ognl/directory-listing.md) evaluation probe, [remote code execution](ognl/remote-code-execution.md), and [remote file inclusion](ognl/remote-file-inclusion.md).
- **[SpEL](spel/index.md)**: the Spring Expression Language, covering [code execution](spel/code-execution.md) and [WAF bypass](spel/waf-bypass.md).

## Tools

- **Burp Suite**: Repeater and Intruder for injecting SpEL, OGNL, MVEL, and unified EL.
- **J2EEScan**: Burp extension with active EL and OGNL injection checks.

## References

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
