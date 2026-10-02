---
title: "Server-side template injection (SSTI): user-controlled template syntax evaluated in the server process"
description: How user input that reaches a template-compilation or expression-evaluation sink crosses from data into template code, letting an attacker walk the engine's object model to reach file read and remote code execution.
keywords:
  - SSTI
  - server-side template injection
  - template injection
  - Jinja2
  - Thymeleaf
  - remote code execution
---

# Server-side template injection

**Server-side template injection (SSTI)** is a vulnerability in which application code places untrusted input into a **template** that the engine then compiles and evaluates in the server process. Template engines are small programming languages: they resolve variables, call methods, and walk object graphs. When the attacker controls part of the template *source* rather than only the *data* fed into a fixed template, their input crosses from the data channel into **code** the engine runs—reaching the host object model, the engine's OS bridges, and ultimately **remote code execution (RCE)** in the context of the application's service account.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Evaluating template payloads against systems without written authorization is unlawful.

## Overview

Applications build templates dynamically for ordinary reasons: a user-customizable email, a themeable dashboard, a wiki macro, a report format, a "personalized" greeting that concatenates a name into a template string. The bug appears when that concatenation reaches an API that compiles template syntax—`render_template_string`, `Template(user_string)`, `new SpelExpressionParser().parseExpression(input)`, a Thymeleaf fragment composed from an HTTP value.

The decisive distinction, which defenders and testers both miss, is **where** the input lands:

- **Template source position (SSTI).** Input is concatenated into the template text the engine *parses*. The engine sees attacker-authored syntax: `{{ 7*7 }}` becomes `49`. This is the exploitable case.
- **Template data position (safe).** Input is passed as a *context variable* to a fixed, file-backed template. `{{ name }}` with `name="{{7*7}}"` renders the literal string `{{7*7}}`—the engine never re-parses it.

Exploitation is a progression: confirm evaluation, **fingerprint** the engine, then climb the engine's exposed objects until a method reaches the filesystem or a subprocess.

## Why it reaches execution

- **Templates are Turing-complete-ish.** Most engines resolve attribute access, index into objects, and call methods. Once an attacker reads one object (`config`, `self`, a class reference), the language's own reflection walks the rest of the runtime.
- **Convenience APIs compile strings.** `render_template_string`, Jinja `Environment.from_string`, Twig `createTemplate`, Django `Template()`—each makes "render this string" one call away from "render this file."
- **Sandboxes are opt-in and leaky.** Jinja2's `SandboxedEnvironment`, Thymeleaf's restricted dialects, and Velocity's secure uberspect block known gadgets but are routinely disabled or bypassed by a new object path.
- **Second-order flows.** A stored profile field or a saved "template" is later rendered elsewhere, so the taint is far from where it entered.

## Impact

Full RCE is the headline outcome, but partial primitives matter: arbitrary file read (configuration, source, secrets), server-side request forgery through objects that fetch URLs, and **blind** execution where no rendered output is reflected. As with command injection, the ceiling is the **service account's privileges** and the host's **network position**—a renderer that can reach an internal metadata endpoint is often worth more than a shell on an isolated box.

## Pages

| Page | Focus |
|------|--------|
| [Jinja2 and Django](jinja2-and-django-templates.md) | Python stacks: detection in `{{ }}` vs `{% %}`, the `__mro__`/`__subclasses__` climb to RCE, Flask `config` leaks |
| [JSP and Thymeleaf](java-server-pages-and-thymeleaf.md) | Java view layers: Thymeleaf expression preprocessing, SpEL/OGNL `T()` reflection, `Runtime.exec` chains |
| [Sandbox escape](ssti-sandbox-escape.md) | Object-graph and gadget-style chains that climb out of a restricted engine to OS primitives |

## References

- [PortSwigger Web Security Academy: Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)
- [OWASP: Server-Side Template Injection Testing](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE-1336: Improper Neutralization of Special Elements Used in a Template Engine](https://cwe.mitre.org/data/definitions/1336.html)
- [CWE-94: Improper Control of Generation of Code (Code Injection)](https://cwe.mitre.org/data/definitions/94.html)
- [PayloadsAllTheThings: Server Side Template Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
