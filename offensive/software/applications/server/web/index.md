---
title: "Web application attack surface"
description: "Where web vulnerabilities live: the platform that hosts the service, the runtime that executes it, and the application code developers wrote."
keywords:
  - web
  - web application security
  - attack surface
  - server
  - application code
---

# Web

A web service is attacked at three layers: the **platform** (the server process and how it is hosted and wired), the **runtime** (the language interpreter or virtual machine), and the **code** the developers wrote. Where a vulnerability lives decides how it is reached and fixed: a flaw in application logic behaves very differently from one that abuses the server process or the interpreter.

Triage by layer: identify which layer owns the faulty configuration, primitive, or logic from the evidence you have (headers, stack traces, configuration, behavior), then navigate into the matching subtree.

## Subtopics

- **[Code](code/index.md)**: Flaws that originate in the application source the development team controls: unsafe query and command composition, broken authorization, and business-logic...
- **[Platform](platform/index.md)**: Offensive techniques against the web platform organized the way an audit works: general exposures that apply to any server, then per-product misconfiguration...
- **[Runtime](runtime/index.md)**: Offensive techniques against the interpreter or VM that runs web application code: insecure deserialization, engine stream wrappers, and escaping runtime san...
