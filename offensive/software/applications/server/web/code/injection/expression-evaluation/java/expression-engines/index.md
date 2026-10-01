---
title: "Java expression engines: injection into embedded evaluation libraries"
description: "Standalone JVM evaluators an application embeds for formulas, rules, and templates, and how attacker-controlled expression text is reached, separated by which engines expose the runtime and which stay sandboxed."
keywords:
  - expression engine injection
  - JEXL injection
  - Seam EL injection
  - Exp4j
  - mXparser
---

# Expression engines

Applications embed a standalone expression evaluator whenever they let users or configuration supply a formula, a filter, a rule, or a computed field rather than a fixed calculation. The library parses the string and evaluates it against a context the host provides. When that string carries untrusted input, the attacker controls what the evaluator runs, and the ceiling depends entirely on what the specific engine can touch.

Two of the engines here reach the JVM runtime and give code execution; two are mathematics-only and do not.

- [Exp4j](exp4j.md): a pure mathematical evaluator. Numbers, operators, and host-registered functions and variables only. No reflection, no types, no runtime, so no inherent code execution.
- [JBoss Seam EL](jboss-seam-el.md): Seam evaluates Unified EL (`#{...}`), and an injected EL expression reaches arbitrary Java, giving code execution.
- [JEXL](jexl.md): Apache Commons JEXL expressions can construct classes and invoke methods, reaching `ProcessBuilder` and the runtime.
- [mXparser](mxparser.md): a scientific math parser with user-defined functions and constants. Like Exp4j it has no runtime access and no inherent code execution.

## References

- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
