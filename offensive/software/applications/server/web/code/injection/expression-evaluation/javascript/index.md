---
title: "Server-side JavaScript expression evaluation and prototype pollution"
description: "Server-side JavaScript evaluation sinks and prototype pollution: untrusted keys poison Object.prototype, changing shared defaults that later drive auth bypass and gadget-chain RCE."
keywords:
  - server-side JavaScript injection
  - prototype pollution
  - Object.prototype
  - Node.js merge
  - gadget chain
---

# JavaScript

Server-side JavaScript runs attacker-reachable values in Node.js, and two classes of flaw matter here. One is direct evaluation, where input reaches a construct that compiles and runs JavaScript. The other is prototype pollution, which never evaluates attacker code directly but corrupts the shared `Object.prototype` so that properties the application reads later take values the attacker chose. The second is the subtler and more common server-side exposure, because the polluting request and the point where the damage is realized are usually far apart in the code.

- [Prototype pollution](prototype-pollution.md) covers the server-side Node case: the deep-merge and recursive-assign sinks, the `__proto__` and `constructor.prototype` polluting payloads, and the escalation from poisoned defaults to authorization bypass and gadget-driven command execution.

Browser-side DOM prototype pollution shares the root cause but has a different sink surface and gadget set, and is covered separately.

## Tools

- **Burp Suite**: Repeater to send polluting JSON and probe downstream gadgets.
- **Server-Side Prototype Pollution Scanner**: PortSwigger Burp extension detecting server-side pollution.

## References

- [OWASP: Prototype Pollution](https://owasp.org/www-community/attacks/Prototype_pollution)
- [PortSwigger: Prototype pollution](https://portswigger.net/web-security/prototype-pollution)
- [MDN: Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)
