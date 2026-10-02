---
title: "Gopher URL scheme for SSRF: line-oriented TCP payloads to internal hosts and scheme allowlist gaps"
description: Using gopher:// URLs so a vulnerable client issues line-oriented TCP payloads to arbitrary hosts and ports.
keywords:
  - SSRF
  - gopher
---

# Gopher

## Context

The gopher protocol sends a small TCP payload after connect. Some URL-fetch libraries enable `gopher://` while policy only mentions `http`. That can reach Redis, SMTP, or other line-based services on internal hosts if the client stack allows it. Use only in isolated labs.

## Theory

Crafting requires URL-encoding newlines and command text in the path. Behavior is highly version-specific (Java, curl, custom wrappers).

## Practice

- In a local lab, point a vulnerable fetch at `gopher://127.0.0.1:6379/...` against a disposable Redis instance and observe whether arbitrary keys can be set.

## Tools

- **Burp Suite**
- Custom harness scripts