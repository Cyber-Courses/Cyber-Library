---
title: "Runtime: attacking the language engine behind a web application"
order: 2
description: "Offensive techniques against the interpreter or VM that runs web application code: insecure deserialization, engine stream wrappers, and escaping runtime sandboxes and function restrictions."
keywords:
  - runtime exploitation
  - insecure deserialization
  - php wrappers
  - disable_functions bypass
  - sandbox escape
---

# Runtime

The **runtime** is the language engine that executes the application: the PHP, Python, Node.js, Java, Ruby, or .NET process that parses and runs developer code. This layer owns a distinct class of vulnerabilities, separate from the application logic above it ([Code](../code/index.md)) and the server platform beneath it (Platform). The test for whether a bug belongs here: *would it still exist if the application source were benign and the front-end server were hardened, because the weakness is in how the engine parses, executes, or bridges to the OS?*

Runtime bugs are overwhelmingly **engine-specific**: a PHP object-injection chain, a Java gadget chain, and a Python `pickle` payload share a name and nothing else. So this area is organized by primitive, then by language, because the gadgets and payloads diverge per engine.

## Where it breaks

- **Deserialization.** Engines that rebuild objects from a byte stream run engine-defined callbacks (magic methods, `__reduce__`, `readObject`) during reconstruction. Feed the sink attacker-controlled bytes and those callbacks become a code-execution chain.
- **Stream wrappers.** Engines expose pseudo-protocols (`php://`, `phar://`, `data://`) that turn an innocuous file operation into source disclosure, remote inclusion, or deserialization.
- **Sandboxes and function restrictions.** Deployments that try to contain the runtime (`disable_functions`, `open_basedir`, a language sandbox, a Node `vm`) rely on boundaries the engine itself can be coaxed to cross.

## Sections

- **[Insecure deserialization](insecure-deserialization/index.md)**: rebuilding attacker-controlled objects into RCE, by language (PHP, Python, Java, Ruby, .NET, Node.js).
- **[File wrappers and stream handlers](file-wrappers-and-stream-handlers/index.md)**: engine stream wrappers that escalate file operations to disclosure, deserialization, and RCE.
- **[Sandbox and function restriction escape](sandbox-and-function-restriction-escape/index.md)**: defeating `disable_functions`, `open_basedir`, and language or VM sandboxes.

## References

- PortSwigger Web Security Academy: Insecure deserialization
- OWASP: Deserialization of untrusted data
