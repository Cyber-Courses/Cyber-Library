---
title: "Dynamic file inclusion: LFI, RFI, and template path injection in server-side code"
description: include(), import, or template partial resolution driven by request parameters, local and remote file inclusion patterns in PHP, Java, and Node stacks.
keywords:
  - LFI
  - RFI
  - local file inclusion
---

# Dynamic file inclusion

## Context

The server maps a user parameter to a **filesystem path** or **module id** for `include`, JSP path, or template name. Without an allowlist, attackers read **sensitive files** or, where **RFI** is enabled, pull remote code (legacy stacks).

## Theory

Wrapper protocols (`php://filter`, `expect://`) are stack-specific; map the runtime.

## Practice

- Probe `?page=`, `?template=`, or JSON `templatePath` with traversal and known file markers in a local replica.

## Tools

- **Burp Suite**
- **LFI wordlists** (lab scope)
