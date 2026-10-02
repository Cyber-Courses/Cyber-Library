---
title: "Python expression and policy-language evaluation injection"
description: "Server-side Python evaluation sinks: eval and exec reach arbitrary code execution, while Common Expression Language stays sandboxed and bends toward policy and logic abuse."
keywords:
  - Python eval injection
  - exec code execution
  - Common Expression Language
  - CEL policy bypass
  - expression evaluation
---

# Python

Python applications evaluate attacker-reachable expressions in two very different places, and the two behave nothing alike once input arrives. The language's own `eval` and `exec` compile and run arbitrary Python, so any input that reaches them is a direct path to code execution on the host. Embedded policy languages such as Common Expression Language are deliberately not Python: they are sandboxed evaluators used for admission control, authorization, and validation, so the same injection mindset produces logic and policy manipulation rather than a shell.

Read both pages together, because the practical mistake is treating one like the other. An expression sink that runs real Python is scored as remote code execution; a CEL expression that an operator believed was "just a filter" is scored as a policy or authorization bypass, not RCE.

- [eval() and exec()](python-eval.md) covers genuine arbitrary code execution through the interpreter, builtins abuse, the class-traversal gadget walk that survives restricted builtins, and why `ast.literal_eval` is the safe sibling.
- [Common Expression Language](common-expression-language.md) covers the sandboxed case: manipulating authorization and admission decisions, disclosing context exposed to the expression, and abusing any dangerous host-registered functions.

## Tools

- **Burp Suite**: Repeater and Intruder for reaching eval/exec and CEL sinks.
- **tplmap**: detects and exploits Python eval()/exec() code-injection sinks.

## References

- [Python: eval built-in](https://docs.python.org/3/library/functions.html#eval)
- [Python: ast.literal_eval](https://docs.python.org/3/library/ast.html#ast.literal_eval)
- [CEL specification](https://github.com/google/cel-spec)
