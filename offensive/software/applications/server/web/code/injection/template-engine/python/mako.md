---
title: "Mako server-side template injection"
description: "Exploiting Mako SSTI: Mako evaluates Python directly inside expression and code blocks, so a template injection is immediate remote code execution."
keywords:
  - Mako SSTI
  - Python template injection
  - direct code execution
  - module-level block
---

# Mako

Mako (used by Pylons, Pyramid, and some standalone apps) differs from Jinja2 in a crucial way: it has no sandbox and evaluates arbitrary Python inside its expression and code constructs. A template injection is therefore direct code execution, with none of the object-graph walking Jinja2 needs.

Confirm with `${7*7}` returning `49`. Expression blocks evaluate any Python expression, so importing and running a command is a one-liner:

```mako
${__import__('os').popen('id').read()}
```

Mako's control blocks run statements, which allows a cleaner import then call:

```mako
<% import os %>${os.popen('id').read()}
```

Module-level blocks (`<%! ... %>`) run once at import time and are equally usable. Mako also exposes helpers whose attributes reach the OS, for example `${self.module.cache.util.os.system('id')}`, but the direct `__import__('os')` form is the simplest and most reliable.

Because evaluation is unrestricted, there is no sandbox to escape; the only obstacles are input filters on the application side (blocking `${`, `import`, or backticks), which are bypassed with Mako's alternative block syntaxes and Python's many ways to obtain a module (`__import__`, `importlib`, `os` via `sys.modules`). The takeaway is that confirming Mako effectively confirms RCE, so detection and exploitation collapse into one step.

## Tools

- tplmap, SSTImap

## References

- Mako documentation: expression and control syntax
- PortSwigger Web Security Academy: Server-side template injection
