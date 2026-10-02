---
title: "Tera server-side template injection"
description: "Exploiting Tera SSTI in Rust: confirming with {{7*7}}, enumerating the render context and built-in filters for information disclosure, and the Tera::one_off sink and its limits."
keywords:
  - Tera SSTI
  - Tera one_off
  - context disclosure
  - Rust template injection
---

# Tera

Tera parses templates at runtime and uses Jinja2-style `{{ }}` expressions and `{% %}` statements. Confirm with `{{7*7}}` rendering `49`. Unlike Jinja2, Tera has no object graph reaching the interpreter: values come only from the render context the application supplies, and callable behavior is limited to registered filters, tests, and functions. There is no attribute walk to a module, no `__globals__`, and no construct that imports or spawns a process.

The realistic impact is information disclosure. Dump what the context exposes and enumerate the built-in filters, which can transform and leak data:

```tera
{{ __tera_context }}
{{ some_var }}
{{ get_env(name="PATH") }}
```

`__tera_context` prints the serialized render context, often revealing variables the page did not intend to show. The built-in `get_env` function reads environment variables when it has not been removed, which can leak secrets and configuration. Registered custom functions are the one escalation path: if the application registered a Tera function that performs a sensitive action (runs a command, reads a file), the template can call it by name with arguments. Enumerate the registered functions and test each.

The sink is `Tera::one_off(user_input, &context, autoescape)` or rendering a template string assembled from user input; templates loaded from the project's own files are not attacker-controlled. Treat Tera as disclosure-only by default, mine `__tera_context` and `get_env` for secrets, and escalate only through an application-registered function with a side effect.

## Tools

- Manual enumeration

## References

- Tera documentation: built-in filters and functions, one_off
- PortSwigger Web Security Academy: Server-side template injection
