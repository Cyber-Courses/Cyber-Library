---
title: "File injection"
description: "Abuse of how a web application takes in, resolves, includes, or exports files, turning file handling into code execution, disclosure, or downstream interpreter attacks."
keywords:
  - file injection
  - file upload
  - local file inclusion
  - remote file inclusion
  - export injection
---

# File

File injection groups the attacks that abuse how a web application takes in, resolves, and emits files rather than how it builds a query.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

Three tracks run through this subtree. **Upload** covers getting attacker-controlled bytes written to a path the server will later execute or serve, including web-shell planting and archive extraction that escapes its target directory. **Inclusion** covers dynamic `include`/`require` style sinks where a path or URL fragment is attacker-controlled, yielding source disclosure, local file reads, and code execution. **Exports** covers the opposite direction: data the application writes out (CSV, LaTeX, PDF) that a downstream program parses and interprets, turning an export feature into command execution or data theft on another system. Each technique page carries concrete, runnable payloads.
