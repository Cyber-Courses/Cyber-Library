---
title: "Server-side template injection (SSTI)"
description: "Exploiting server-side template injection: detecting the engine, escalating from a template expression to the host language runtime, and the sandbox limits that decide whether RCE is reachable, organized by language."
keywords:
  - server-side template injection
  - SSTI
  - template engine
  - sandbox escape
  - RCE via template
---

# Template Engine

Server-side template injection (SSTI) happens when user input is embedded into a template that the server then renders, so the input is evaluated as template code rather than data. Because templates can reach into the host language, SSTI often escalates from reflected output to remote code execution, with the exact ceiling set by the engine and its sandbox.

The first step is detecting the engine, because each has its own expression syntax. A mathematical probe that renders differently per engine narrows it down: `{{7*7}}` (returns `49`) points at Jinja2, Twig, or Nunjucks; `${7*7}` at FreeMarker or a JSP/EL context; `#{7*7}` at Pug or a JSF context; `<%= 7*7 %>` at ERB; `#set($x=7*7)$x` at Velocity; `@(7*7)` at Razor; `[% 7*7 %]` at Template Toolkit. A polyglot such as `${{<%[%'"}}%\` triggers a revealing error in several engines. When `7*7` renders as `49` the input is being evaluated, not just reflected.

The escalation pattern is shared: from a template expression, walk the object graph or built-ins the engine exposes to reach the language runtime (a class loader, a globals dictionary, a utility that executes commands), then call out to the OS. Whether that succeeds depends on the sandbox: some engines run untrusted templates in a restricted mode by default (and the task is to escape it), while others, and most engines when the framework does not enable sandboxing, evaluate freely.

This area is organized by language, then by engine, since the gadgets are engine-specific.

## Languages

- **[Python](python/index.md)**: Jinja2, Mako.
- **[PHP](php/index.md)**: Twig, Smarty.
- **[Java](java/index.md)**: FreeMarker, Velocity.
- **[JavaScript](javascript/index.md)**: Handlebars, Pug.
- **[Ruby](ruby/index.md)**: ERB.
- **[Go](go/index.md)**: text/template and html/template.
- **[C#](c-sharp/index.md)**: Razor.
- **[Rust](rust/index.md)**: Tera.
- **[Perl](perl/index.md)**: Template Toolkit.

## References

- PortSwigger Web Security Academy: Server-side template injection
- OWASP Testing Guide: Testing for Server-Side Template Injection
