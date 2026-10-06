---
title: "Desktop: attacking installed desktop applications"
description: "Offensive techniques against desktop applications: local interprocess communication and custom protocol handlers, insecure auto-update channels, embedded-browser and Electron weaknesses that reach the operating system, deserialization and parsing flaws, and secrets the application stores on the endpoint."
keywords:
  - desktop application security
  - Electron
  - protocol handler
  - insecure update
  - local IPC
---

# Desktop

A desktop application runs with the user's own privileges and has direct local reach: the filesystem, other processes, and the operating system. Its attack surface is what it exposes locally to other software, how it fetches and trusts updates and content, and the secrets it keeps on disk. The attacker is often another local process or a web page that can reach the app through a registered handler.

## The surface

- **Local IPC and protocol handlers**: named pipes, local sockets, and custom URL-scheme handlers that other local software (or a web page) can invoke to drive the application.
- **Insecure updates**: auto-update channels that fetch over weak transport or without signature verification, turning the updater into code execution.
- **Embedded browsers and Electron**: applications built on web runtimes where disabled isolation, exposed Node integration, or a navigation to attacker content reaches the operating system.
- **Parsing and deserialization**: file and message formats the application opens, and the memory-safety or deserialization flaws in parsing them.
- **Local secrets**: credentials, tokens, and keys the application stores on the endpoint.

## References

- [Electron security guidelines](https://www.electronjs.org/docs/latest/tutorial/security)
- [OWASP Desktop App Security Testing](https://owasp.org/www-project-desktop-app-security-testing/)
