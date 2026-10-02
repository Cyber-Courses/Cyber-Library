---
title: "SpEL injection: Spring Expression Language abuse"
description: "Spring evaluates SpEL from request data through SpelExpressionParser, and with the default evaluation context a type reference reaches Runtime and ProcessBuilder for code execution."
keywords:
  - SpEL injection
  - Spring Expression Language
  - SpelExpressionParser
  - "T() type reference"
  - SpEL RCE
---

# SpEL

The Spring Expression Language (SpEL) evaluates expressions against an object graph at runtime. It appears throughout Spring: `@Value` annotations, Spring Security access rules, Spring Integration routing, Spring Data projections, and any code that calls a `SpelExpressionParser` directly. When request data reaches the string parsed into a SpEL expression, through a reflected parameter, a user-supplied rule, or a template value, the SpEL grammar runs inside the application.

SpEL is a full object language. The `T(...)` operator is a type reference that names any class on the classpath, and from there the expression calls static methods, constructs objects, and invokes instance methods. Under a `StandardEvaluationContext`, the default when code builds its own parser, that reaches `java.lang.Runtime` with no restriction. A `SimpleEvaluationContext` restricts the grammar, so the reachable surface depends on which context the application supplied.

- **[Code execution](code-execution.md)**: the type reference and reflection routes from a SpEL expression to the host runtime.
- **[WAF bypass](waf-bypass.md)**: rewriting the code-execution payload to defeat signature filters while keeping it valid SpEL.

## Tools

- **Burp Suite**: Repeater and Intruder for SpEL injection into Spring sinks.
- **J2EEScan**: Burp extension with SpEL injection checks.

## References

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
