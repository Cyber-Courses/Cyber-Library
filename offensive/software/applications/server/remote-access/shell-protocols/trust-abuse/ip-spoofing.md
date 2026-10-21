---
title: "IP spoofing: impersonating a trusted source for the r-commands"
order: 1
description: "Trust-file-based r-command authentication considers the remote host and username and traditionally expects a privileged client source port. An on-path attacker may impersonate a trusted endpoint and observe the exchange; a blind attacker additionally needs predictable TCP sequence numbers and protocol-correct packet injection."
keywords:
  - ip spoofing
  - source address
  - tcp sequence prediction
  - trusted host
  - blind spoofing
---

# IP spoofing

Trust-file-based services such as `rsh`, `rlogin`, and `rcp` authenticate a claimed remote host and username against `/etc/hosts.equiv` or the target account's `.rhosts`; traditional implementations also expect the client to use a privileged source port. An attacker therefore has to satisfy the complete host, user, account, and protocol checks—not merely place a trusted address in an IP header. With an on-path position, an attacker may be able to impersonate a trusted endpoint while observing and manipulating the TCP exchange. Blind exploitation is substantially harder: it requires forging the trusted source, predicting the server's TCP sequence state, and injecting protocol-correct data without seeing replies. `rexec` is separate because it authenticates with a username and password rather than these trust files.

```bash
# identify a trusted host to impersonate (from a readable hosts.equiv/.rhosts)
cat /etc/hosts.equiv ~*/.rhosts 2>/dev/null
# verify the exact remote-host/remote-user pair, requested local account,
# privileged source-port requirement, and any hostname checks used by the target
# on-path: intercept or forge a protocol-correct exchange while observing TCP state
# blind: predict TCP state and inject a complete protocol request from the trusted address
# a successful rsh command might plant .rhosts to convert temporary access into persistence
```

## Exploitation notes

- The usual objective is to inject one command that plants durable trust ([rhosts write](rhosts-write.md)), after which the attacker connects normally without spoofing.
- An on-path position provides visibility into TCP state, but the attacker must still satisfy the service's source-port, username, target-account, and trust-rule checks. Blind spoofing additionally needs predictable TCP sequence numbers, which modern stacks randomise, so it is viable mainly against legacy systems.
- A successful attack may not require a password or prior file-write primitive, but it does require a matching trusted host/user identity and a protocol-correct connection. Hostname-resolution policy and implementation-specific checks can add further constraints.
- The specific trusted address to forge comes from reading `hosts.equiv`/`.rhosts` or inferring the trust relationships.

## References

- [RFC 1948 (defending against sequence prediction)](https://www.rfc-editor.org/rfc/rfc1948)
- [Linux rcmd/ruserok authentication routines](https://man7.org/linux/man-pages/man3/rcmd.3.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
