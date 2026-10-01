---
title: "File inclusion"
description: "Dynamic include and require sinks where an attacker-controlled path or URL fragment selects what the interpreter loads, yielding disclosure or code execution."
keywords:
  - file inclusion
  - LFI
  - RFI
  - include
  - require
  - php wrappers
---

# Inclusion

Inclusion flaws live in dynamic `include`/`require` style sinks, where a request parameter decides which file the interpreter loads and runs.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

Classic PHP code like `include($_GET['page'] . '.php')` hands the attacker control over the loaded path. Two directions follow. **Local** inclusion points the sink at files already on the server: traversal to read source and secrets, wrappers to disclose or decode, and poisoning tricks (logs, `/proc`, session files, prior uploads) that convert a read primitive into code execution. **Remote** inclusion points the sink at an attacker-hosted URL so the interpreter fetches and runs external code directly, subject to the engine's URL-include settings. The two pages cover the payloads and the chains that promote a mere file read to full execution.
