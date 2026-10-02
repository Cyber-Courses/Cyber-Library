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

Three tracks run through this subtree. **Upload** covers getting attacker-controlled bytes written to a path the server will later execute or serve, including web-shell planting and archive extraction that escapes its target directory. **Inclusion** covers dynamic `include`/`require` style sinks where a path or URL fragment is attacker-controlled, yielding source disclosure, local file reads, and code execution. **Exports** covers the opposite direction: data the application writes out (CSV, LaTeX, PDF) that a downstream program parses and interprets, turning an export feature into command execution or data theft on another system. Each technique page carries concrete, runnable payloads.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater and Intruder for upload, inclusion, and export payloads.
- **[fimap](https://github.com/kurobeats/fimap)**: automated file-inclusion testing.
- **[exiftool](https://exiftool.org/)**: embed payloads in file metadata for polyglots.

## References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
