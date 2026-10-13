---
title: "Go server-side template injection"
order: 1
description: "SSTI in Go's text/template and html/template: what the engine does and does not expose, method calls on injected data, and why command execution usually is not reachable from the template alone."
keywords:
  - Go SSTI
  - text/template
  - html/template
  - method call injection
---

# Go

Go's standard templating is `text/template` and its auto-escaping sibling `html/template`. Both are deliberately restricted: a template can reference fields and call methods on the data passed to it, but it cannot import packages, construct arbitrary types, or reach the runtime. This makes Go an important honest case: injecting template syntax rarely yields command execution the way Jinja2 or FreeMarker do.

## Engines

- **[text/template and html/template](go-templates.md)**: detection, field and method enumeration, information disclosure, and the method-call conditions under which impact escalates.

## References

- Go documentation: text/template, html/template
- PortSwigger Web Security Academy: Server-side template injection
