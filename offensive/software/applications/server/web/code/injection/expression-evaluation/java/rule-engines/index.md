---
title: "Java rule engines: injection into routing, workflow, and pipeline expressions"
order: 2
description: "Rule, routing, workflow, and log-pipeline products evaluate expressions and consequences over their input, and attacker-controlled expression text reaches the underlying language and the JVM runtime."
keywords:
  - rule engine injection
  - Drools MVEL
  - Camel Simple language
  - Camunda JUEL
  - Logstash ruby filter
---

# Rule engines

Rule, routing, workflow, and pipeline products let operators express logic as data: a Camel route condition, a Drools rule consequence, a Camunda process expression, a Logstash filter. Each ships its own expression or scripting layer, and each evaluates that layer against the messages or events flowing through it. When any part of that expression is built from untrusted input, the attacker controls the logic the engine runs, and on these engines that logic reaches a full language with host access.

Every engine in this group reaches code execution, though through a different language:

- [Apache Camel](apache-camel.md): the Simple language (`${...}`) invokes beans and methods, and routes using the OGNL or SpEL components reach method calls and the runtime.
- [Camunda](camunda.md): BPMN uses Unified EL (JUEL) `${...}` bound to Java methods, and script tasks run Groovy or JavaScript directly.
- [Drools](drools.md): rule conditions and consequences use the MVEL dialect, which has full Java access.
- [ELK Logstash](elk-logstash.md): the pipeline `ruby` filter executes arbitrary Ruby supplied in its `code` option.

The JVM engines here share the `Runtime.exec(String)` shell caveat: the single-string form tokenizes on whitespace with no shell, so shell features require an explicit `String[]{"/bin/bash","-c","..."}` vector through `ProcessBuilder`. The Logstash page covers the Ruby equivalent.

## Tools

- **Burp Suite**: Repeater and Intruder for injecting into route, rule, and pipeline expressions.
- **J2EEScan**: Burp extension with checks for Java EL and OGNL injection.

## References

- [Apache Camel: expression languages](https://camel.apache.org/components/latest/languages/index.html)
- [Drools documentation](https://docs.drools.org/)
- [Camunda: expression language](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)
