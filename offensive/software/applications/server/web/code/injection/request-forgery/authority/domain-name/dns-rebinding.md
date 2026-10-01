---
title: "DNS rebinding to defeat SSRF host allowlists"
description: "A hostname that resolves to a public IP at validation time and an internal IP at connection time lets one allowed name reach loopback or metadata through a time-of-check to time-of-use gap in resolution."
keywords:
  - DNS rebinding
  - TOCTOU resolution
  - short TTL
  - metadata IP
  - host allowlist bypass
  - resolve twice
---

# DNS rebinding

Host-based SSRF defenses resolve the supplied hostname, check the resulting IP against a policy, and allow the request if it looks external. DNS rebinding breaks that by making the name resolve to *different* addresses at check time and at connection time. The validator sees a public IP and approves; moments later the client resolves the same name again and connects to an internal one.

## The time-of-check gap

The weakness is two separate resolutions of one hostname: once by the validator, once by the HTTP client when it opens the socket. If the authoritative server the attacker controls returns a different answer for the second lookup, the policy decision no longer matches the connection.

The attacker serves `attacker.example` from a nameserver they run and answers with a very short TTL so the resolver does not cache:

```
attacker.example.  1  IN  A  203.0.113.10     # public, returned at validation
```

Once the application has validated and cached its allow decision, the next query for the same name is answered with an internal address:

```
attacker.example.  1  IN  A  127.0.0.1        # loopback, returned at connect
```

The short TTL (often `0` or `1`) forces the client to re-resolve between the check and the connect, so the second answer is the one the socket uses. The target is typically `127.0.0.1`, an RFC1918 host, or the metadata address `169.254.169.254`.

## Multiple-answer and fast-flip variants

Some stacks resolve only once, so a pure check-then-connect flip does not apply. Two variants cover them:

- **Multiple A records**: return both a public and an internal address in one response and let the client try them in turn. A validator that inspects only the first record passes, while the client may connect to the second.
- **Rapid re-answer**: alternate answers on a timer so that whichever resolution the client performs after validation lands on the internal address. Tooling that automates this flip (serving public, then private, keyed on request count or time) makes the race reliable.

## Browser-driven fetches

When the fetch is performed by a [headless browser](../../fetch/headless-browser.md), rebinding also crosses the browser's own network boundary: a page loaded from the attacker's public address later rebinds to an internal IP while keeping the same origin, so script on the page reads internal responses. Private Network Access checks that gate on the document's apparent address are satisfied by the first, public answer.

## Why it beats allowlists specifically

An allowlist of permitted *hostnames* is the ideal target: the name stays constant and allowed, only its address changes. A control that resolves the name once, pins that exact IP, and connects to the pinned address closes the gap, which is why rebinding works against name-based checks but not against connect-to-the-validated-IP designs. When the application resolves only once and pins, move to an [internationalized domain](internationalized-domain.md) or raw [IP address](../ip-address.md) approach instead.

## Tools

- [Singularity of Origin](https://github.com/nccgroup/singularity) (DNS rebinding attack framework)
- [rebind / rbndr](https://github.com/taviso/rbndr) (fast-flip rebinding name server)

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
