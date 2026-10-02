---
title: "Software"
description: Offensive security topics that target software—applications, services, and the code that implements them—across client and server roles.
keywords:
  - offensive software security
  - application security
  - server-side testing
  - vulnerability research
---

# Software

Offensive work in this branch focuses on **software** as the primary attack surface: programs, libraries, APIs, and the business and security rules implemented in code. It complements other offensive areas (for example, network or physical testing) by asking what can go wrong when untrusted input meets application logic, data stores, and access control.

## Organization

- **[Applications](applications/index.md)** — How client and server application types are split in this library. Server-side **web** material lives under *Applications → Server → Web → Code*.

## Scope

Use this subtree when the finding depends on **how the application is built** (logic, framework use, ORM queries, API design) rather than on generic host or network configuration alone. When in doubt, follow the path that matches how an operator would navigate from “HTTP service” down to “a specific handler or query.”

## See also

- [Offensive Security](../index.md) — Top-level introduction to offensive practice in this library.