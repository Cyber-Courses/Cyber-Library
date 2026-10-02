---
title: "JavaScript server-side template injection"
description: "SSTI in Node.js template engines: Handlebars prototype/constructor escapes and Pug's inline JavaScript to reach require('child_process')."
keywords:
  - JavaScript SSTI
  - Node.js template injection
  - Handlebars
  - Pug
---

# JavaScript

Node.js SSTI reaches code execution through `require('child_process').execSync` (or `process` / `global`), since a template that can run JavaScript can load any core module. The engines differ in how much JavaScript they let the template express.

Pug compiles templates to JavaScript and allows inline JS in several constructs, so injection is close to direct evaluation. Handlebars is logic-light by design and exposes no direct JS, so its exploitation is a constructor/prototype walk that reassembles a function at runtime. Detection differs too: Pug responds to `#{7*7}`, while Handlebars uses `{{ }}` but will not evaluate `{{7*7}}` as arithmetic, so it is fingerprinted by its error messages and helper syntax.

## Engines

- **[Handlebars](handlebars.md)**: the `constructor.constructor` / `#with` prototype walk to a function.
- **[Pug](pug.md)**: inline JavaScript in interpolation and buffered code to `require('child_process')`.

## References

- Handlebars and Pug documentation
- PortSwigger Web Security Academy: Server-side template injection
