---
title: "Python server-side template injection"
description: "SSTI in Python template engines: Jinja2 (Flask) object-graph walks to RCE and its sandbox, and Mako's direct Python evaluation."
keywords:
  - Python SSTI
  - Jinja2
  - Mako
  - Flask template injection
  - sandbox escape
---

# Python

Python SSTI is common because Flask ships Jinja2 and `render_template_string` is frequently fed user input. The two engines here differ sharply in how hard RCE is.

Jinja2 restricts direct attribute access in its sandboxed mode but exposes a rich object graph, so exploitation is about walking from an exposed object to the Python runtime. Mako places no such restriction: it evaluates Python directly inside `${ ... }`, so RCE is immediate. Detection for both starts with `{{7*7}}` (Jinja2) or `${7*7}` (Mako) returning `49`.

## Engines

- **[Jinja2](jinja2.md)**: object-graph walks (`cycler`, `config`, `lipsum`) to `os`, and sandbox escape.
- **[Mako](mako.md)**: direct Python evaluation in `${ }` and `<% %>` blocks.

## References

- Jinja2 and Mako documentation (sandboxing, expression syntax)
- PortSwigger Web Security Academy: Server-side template injection
