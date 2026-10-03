---
title: "Network services"
description: "Offensive scope for the network-facing services and RPC interfaces that Windows and infrastructure hosts expose, where a single reachable service can coerce authentication or run code, with the Windows Print Spooler as a long-standing example."
keywords:
  - network services
  - RPC
  - print spooler
  - coercion
  - Windows services
---

# Network services

Windows and infrastructure hosts expose a wide range of network-reachable services and RPC interfaces, and many of them were designed for a trusted LAN rather than a hostile one. A single reachable service can be enough to **coerce a machine into authenticating** to an attacker, or to **load code as SYSTEM**, which is why these interfaces are a staple of internal compromise.

## Services

- **[Print Spooler](print-spooler/index.md)**: the Windows print service (MS-RPRN / MS-PAR), source of both the PrinterBug coercion primitive and the Point and Print driver-loading code execution.

## References

- [Microsoft: MS-RPRN, Print System Remote Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rprn/d42db7d5-f141-4466-8f47-0a4be14e2fc1)
- [HackTricks: print spooler service abuse](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/printers-spooler-service-abuse.html)
