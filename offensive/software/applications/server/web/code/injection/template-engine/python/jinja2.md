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

Two distinct obstacles are often conflated. The first is an application input filter or WAF that blocks the literal strings (`__class__`, `os`, backticks). These are defeated with obfuscation that does not change what the sandbox sees: the `|attr()` filter with an encoded name (`|attr('\x5f\x5fclass\x5f\x5f')`), smuggling the string from a request parameter (`request.args.c`), or concatenating it from pieces. This obfuscation only helps against filtering: in a default (non-sandboxed) Flask environment it lets the globals walk above through.

The second obstacle is a real `SandboxedEnvironment`, and the obfuscation above does not beat it. The sandbox decodes the attribute name and runs it through `is_safe_attribute`, so `|attr('\x5f\x5fclass\x5f\x5f')` resolves to `__class__` and is rejected exactly like the plain form, returning undefined or raising `SecurityError`; `__getitem__` is checked the same way. A genuine sandbox escape therefore does not come from encoding tricks but from a flaw in the sandbox of a specific Jinja2 version: historically the reachable `str.format`/`format_map` methods leaked format-string access to the object graph, and similar method-level gaps were patched over time. Test `{{ ''.__class__ }}` first: if it is rejected, the sandbox is on, and the task is finding a version-specific method the sandbox still allows, not re-encoding the underscores.

## Tools

- tplmap, SSTImap

## References

- Jinja2 documentation: sandbox, template globals
- PortSwigger Web Security Academy: Server-side template injection
