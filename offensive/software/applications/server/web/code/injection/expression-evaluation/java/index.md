---
title: "Java expression-language injection: engines, rule systems, and web frameworks"
description: "How attacker-controlled text reaching a JVM expression language turns an application feature into method invocation and, on the engines with runtime access, code execution."
keywords:
  - Java expression injection
  - expression language
  - EL injection
  - rule engine injection
  - JVM code execution
---

# Java

The JVM hosts a large family of expression languages. Some are embedded directly in frameworks (Unified EL in Jakarta/BPMN, SpEL in Spring), some are standalone evaluation libraries an application pulls in to let users write formulas or rules (JEXL, MVEL, Exp4j, mXparser), and some drive routing or rule products (Camel Simple, Drools, Camunda). They share a common exposure: when application code builds an expression string from untrusted input and hands it to an evaluator, the attacker writes the program the evaluator runs.

The reachable impact is not uniform, and getting it wrong wastes an engagement. The languages split into two groups:

- **Runtime-reaching languages** resolve types, construct objects, and call methods. An injected expression reaches `java.lang.Runtime`, `ProcessBuilder`, and reflection, so the result is code execution in the application process. EL (JUEL/Seam), SpEL, OGNL, MVEL (and Drools built on it), JEXL, Camel Simple, and Camunda JUEL all belong here.
- **Sandboxed math evaluators** parse numbers, operators, and only the functions and variables the host registered. Exp4j and mXparser have no path to Java types, reflection, or the runtime, so they give no inherent code execution. Their injectable surface is the math grammar, any custom functions or variables the application exposed, and resource-exhaustion abuse. Treating these as RCE sinks is a mistake.

Across every runtime-reaching JVM engine the command sink is the same, and so is its one sharp edge. `Runtime.exec(String)` splits its argument on whitespace and runs the result directly with no shell, so `$(...)`, backticks, pipes, `>`, and `&&` are passed as literal arguments and never expand. Any payload that needs shell features builds an explicit argument vector instead:

```java
new String[]{"/bin/bash","-c","id > /tmp/o 2>&1"}
```

and runs it through `ProcessBuilder` or the `Runtime.exec(String[])` overload. The pages below apply this in each engine's native syntax.

This section is organized into three child groups:

- [Expression engines](expression-engines/index.md): standalone evaluation libraries an application embeds for formulas, templates, or scripting: Exp4j, JBoss Seam EL, JEXL, and mXparser.
- [Rule engines](rule-engines/index.md): rule, routing, and workflow products whose expressions and consequences execute on untrusted input: Apache Camel, Camunda, Drools, and ELK Logstash.
- Web frameworks: the expression surfaces embedded in server-side MVC and view layers.

## Subtopics

- **[Web frameworks](web-frameworks/index.md)**: Java web frameworks evaluate expression language drawn from request data, from the unified EL of JSP and JSF to the framework engines SpEL, OGNL, and MVEL th...

## Tools

- **Burp Suite**: Repeater and Intruder for injecting EL, OGNL, and MVEL payloads.
- **J2EEScan**: Burp extension with active checks for Java EL and OGNL injection.

## References

- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
