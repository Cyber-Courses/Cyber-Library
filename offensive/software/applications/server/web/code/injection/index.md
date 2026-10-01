---
title: "Injection vulnerabilities"
description: "Untrusted data bound into something the server executes, parses, stores, renders, or requests, organized by the dangerous primitive under attack."
keywords:
  - injection
  - command injection
  - SQL injection
  - NoSQL injection
  - code injection
---

# Injection

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

Injection is where untrusted data is bound into something the server **executes, parses, stores, renders, or requests**, a database query, an operating-system command, markup, a template runtime, an HTTP message, or an outbound URL. When the data crosses from the *data* plane into the *code* plane, the attacker controls part of what the interpreter does.

This tree is organized strictly by the **primitive under attack**, not by how the input arrives (query string, header, or body, that axis lives under input delivery). Each child names the primitive: database queries, OS commands, markup parsers, template engines, HTTP handling, outbound requests, and more.
