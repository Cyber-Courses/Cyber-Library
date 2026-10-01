---
title: "Expression evaluation injection"
description: "When an application evaluates an attacker-influenced expression string at runtime, the expression language becomes a code path. Organized by language and engine, from math evaluators to full EL interpreters that reach the host runtime."
keywords:
  - expression language injection
  - EL injection
  - SpEL
  - OGNL
  - runtime expression evaluation
  - code execution
---

# Expression evaluation

Expression evaluation injection happens when application code takes a string of untrusted input and hands it to an expression engine that evaluates it at runtime. The engine is meant to compute a formula, filter a rule, or resolve a template variable, but when the string itself is attacker-controlled, its grammar becomes an execution surface. Depending on the engine, that ranges from arithmetic-only evaluation to full access to the host runtime, which on the JVM and in scripting languages means remote code execution.

The sink is a call that compiles and runs an expression: a Spring `SpelExpressionParser`, a Struts OGNL evaluation, a PHP `eval`, a Python `eval`/`exec`, a rules engine resolving a condition, or a data-binding tag that evaluates a property path. Each takes a string and runs it, and each is only as safe as the language it exposes and the sandbox around it.

## Organized by language and engine

The subtrees group by the language the application runs, because the reachable payloads and the escalation to code execution are language-specific:

- **[C#](c-sharp/index.md)**: .NET expression and formula evaluators (NCalc, Flee) and ASP.NET data-binding evaluation.
- **[Go](go/index.md)**: the `expr` expression package and similar mini-languages.
- **[Java](java/index.md)**: the richest surface, with full EL interpreters (SpEL, OGNL, MVEL, JEXL), the JSP and JSF unified EL, and rule engines (Drools, Camel, Camunda). Most reach `java.lang.Runtime` and give code execution.
- **[JavaScript](javascript/index.md)**: server-side evaluation and prototype pollution of shared objects.
- **[PHP](php/index.md)**: `eval` and dynamic code paths.
- **[Python](python/index.md)**: `eval`/`exec` misuse and policy languages such as CEL.

## What the engine decides

Two properties of the engine decide the outcome. First, what the language can reach: a pure math evaluator confined to numbers is a weaker target than an EL that can name arbitrary classes and call methods. Second, whether a sandbox constrains it, and whether that sandbox can be escaped, since several engines ship a restriction that reflection or a type reference walks around. Each page maps its engine's parse and evaluation entry point, what the expression grammar reaches, and the route from a benign-looking formula to the engine's maximum impact.

Regular-expression engine abuse (catastrophic backtracking, user-controlled patterns) is a related but separate concern and lives in its own cross-language pages rather than here, and template-file rendering (SSTI) lives under Template Engine.

## References

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [OWASP WSTG: Testing for Code Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_Code_Injection)
- [PayloadsAllTheThings: Server Side Template Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
