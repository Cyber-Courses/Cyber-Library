---
title: "File inclusion"
order: 2
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

Classic PHP code like `include($_GET['page'] . '.php')` hands the attacker control over the loaded path. Two directions follow. **Local** inclusion points the sink at files already on the server: traversal to read source and secrets, wrappers to disclose or decode, and poisoning tricks (logs, `/proc`, session files, prior uploads) that convert a read primitive into code execution. **Remote** inclusion points the sink at an attacker-hosted URL so the interpreter fetches and runs external code directly, subject to the engine's URL-include settings. The two pages cover the payloads and the chains that promote a mere file read to full execution.

## Pages

- **[Local](local.md)**: LFI through traversal and PHP wrappers for source disclosure, plus log, environ, session, and upload poisoning chains that escalate a file read into code exe...
- **[Remote](remote.md)**: RFI points an include sink at an attacker-controlled URL so the interpreter fetches and executes external code, gated by allow_url_include and allow_url_fopen.

## Tools

- **[LFISuite](https://github.com/D35m0nd142/LFISuite)**: automated LFI detection and exploitation.
- **[kadimus](https://github.com/P0cL4bs/Kadimus)**: LFI and RFI scanning and exploitation.
- **[fimap](https://github.com/kurobeats/fimap)**: automated local and remote file-inclusion testing.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater for traversal, wrapper, and URL-include payloads.

## References

- [OWASP: Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
