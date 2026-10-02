---
title: "Node.js deserialization: node-serialize and function-revival libraries"
description: "Exploiting JavaScript deserialization: node-serialize's function revival as an immediate RCE primitive via an IIFE, plus funcster and serialize-javascript sinks."
keywords:
  - node-serialize
  - javascript deserialization
  - IIFE
  - unserialize
  - funcster
---

# Node.js deserialization

JavaScript has no built-in object-serialization format like PHP or Java, so Node deserialization bugs live in third-party libraries that serialize and revive **functions**. The flagship is `node-serialize`: its `unserialize()` rebuilds a serialized function and, through an immediately-invoked function expression, runs it at load time, making the sink a direct RCE primitive.

## node-serialize

`node-serialize` encodes functions with a `_$$ND_FUNC$$_` marker. On `unserialize()`, a value tagged as a function is passed through `eval`. Appending `()` to the function body turns it into an IIFE that executes during deserialization:

```json
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('id',function(e,o){console.log(o)});}()"}
```

The trailing `()` before the closing quote is the trigger: the revived function invokes itself immediately. Anything calling `serialize.unserialize(userInput)` on this runs the command. `require('child_process')` is reachable because the eval runs in module scope here (unlike a bare `Function` constructor, which is global scope).

Deliver the JSON wherever the app deserializes it: cookies, bodies, cache entries, or queue messages.

## Other libraries

- **`funcster`**: reconstructs functions in a sandboxed module; escape patterns reach `this.constructor.constructor('return process')()` to regain `process` and `require`.
- **`serialize-javascript`**: primarily a serializer, but round-tripping its output through `eval`-based revival in application code reintroduces the function-execution sink.
- **`cryo`, `node-cryo`**: object graph serializers that can revive functions.

## Exploitation notes

- Confirm the library and that your input reaches its `unserialize`/revive call; a plain `JSON.parse` is not vulnerable (it never revives functions).
- For output, `child_process.exec`'s callback does not return to you over HTTP; prefer an OOB callback (DNS/HTTP) or a reverse shell.
- If the revival uses the `Function` constructor (global scope) rather than `eval` (module scope), reach a loader via `process.mainModule.require('child_process')` instead of a bare `require`.

## Tools

- Hand-crafted `_$$ND_FUNC$$_` payloads.

## References

- PortSwigger Web Security Academy: Insecure deserialization
- node-serialize advisory writeups
