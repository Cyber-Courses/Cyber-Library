---
title: "SSTI sandbox escape patterns: object graph, gadget chains, and template-specific primitives"
description: Moving from template data to host objects and code execution in server-side template engines—context-dependent escape primitives.
keywords:
  - SSTI
  - sandbox escape
---

# Sandbox escape (SSTI)

## Context

After proving **code execution** or **file read** is possible, the offensive task is to find the **shortest** chain from the template language’s exposed objects to OS primitives. Names differ per engine (`__mro__`, `getClass`, `T(java.lang.Runtime)`).

## Theory

Engines differ in **auto-escaping**, **sandbox** modes, and **import** restrictions; the same payload string fails across Jinja2 vs Thymeleaf vs Velocity.

## Practice

- Maintain a **per-engine** cheat sheet in lab notes; never paste production secrets into public payload databases.

## Tools

- **tplmap** (where applicable and authorized)
- Engine documentation for the pinned version
