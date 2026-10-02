---
title: "PHP expression evaluation injection"
description: "PHP exposes dynamic-evaluation sinks like eval() that execute arbitrary PHP in the interpreter process, turning attacker-controlled expression strings into direct remote code execution."
keywords:
  - PHP expression injection
  - eval
  - create_function
  - assert
  - dynamic code evaluation
  - remote code execution
---

# PHP

PHP sits at the opposite end of the range from a sandboxed evaluator. Its dynamic-evaluation sinks run arbitrary PHP in the interpreter process, so when attacker input reaches one of them the result is direct, unrestricted remote code execution. There is no grammar to escape and no type surface to enumerate: injected PHP simply runs, with the privileges and the extension set of the running process.

The primary sink is `eval()`, which executes a string as PHP. The same class of exposure comes from `create_function` (which builds a function body from a string) and from `assert` when it is handed a string argument, both of which evaluate their content as code. Each takes a string and runs it, which is why they belong here rather than with template rendering.

## Engines

- **[eval()](eval.md)**: the `eval()` language construct and its sibling dynamic-evaluation sinks, and the path from an injected fragment to command execution.

## Why the impact is maximal

Because the injected string is executed as PHP, the attacker inherits the entire language and runtime: filesystem functions, process execution via `system`, `exec`, `shell_exec`, and the backtick operator, network functions, and whatever extensions and credentials the process holds. Unlike the .NET and Go evaluators elsewhere in this section, there is no narrower ceiling to describe. The only variables are what the PHP process can do on the host and how the input reaches the sink.

## Tools

- **Burp Suite**: Repeater and Intruder for reaching eval, assert, and create_function sinks.
- **tplmap**: detects and exploits PHP eval()-based code injection.

## References

- [PHP manual: eval](https://www.php.net/manual/en/function.eval.php)
- [OWASP: Code Injection](https://owasp.org/www-community/attacks/Code_Injection)
