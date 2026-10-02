---
title: "Edge header trust: forged X-Forwarded and Host accepted as authoritative"
description: "Exploiting proxy wiring that makes client-supplied X-Forwarded-* and Host headers authoritative at the origin: IP spoofing for access and rate limits, and Host-based routing to internal apps."
keywords:
  - x-forwarded-for
  - host header
  - edge header trust
  - internal routing
  - ip spoofing
---

# Edge header trust

A reverse proxy is supposed to *set* trustworthy forwarding metadata (the real client IP, the original scheme and host) and *strip* any client-supplied copies. When the wiring is wrong, the origin trusts headers the client fully controls, so an attacker forges identity, source address, and routing. This page is the edge-wiring angle; the application-side consequences live in [Code > Access Control](../../code/access-control/trust-boundary/proxy-header-trust-abuse.md) and [Identification](../../code/identification/rate-limits-and-automation.md).

## Forged X-Forwarded-* for IP-based decisions

If the proxy appends to (rather than replaces) `X-Forwarded-For`, or if the origin reads the *first* value, or if the origin is reachable directly, a client-supplied `X-Forwarded-For` is believed:

```
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: 10.0.0.5, <real>
X-Real-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
```

Uses: reach IP-gated admin paths ("allow 127.0.0.1/internal only"), defeat per-IP rate limits and lockouts by rotating the header per request, and spoof geo/allowlist checks. The reliability depends entirely on the proxy chain, so test which header and which position (first vs last) the origin honors.

## Forged scheme and host

- `X-Forwarded-Proto`/`X-Forwarded-Host` that the app uses to build absolute URLs enable cache and redirect issues and feed [password-reset poisoning](../../code/authentication/password-reset-and-invite-tokens.md).
- `X-Forwarded-Port` and similar influence link generation and some access checks.

## Host-based routing to internal apps

Many deployments route by `Host` at the proxy. Supplying an internal hostname can reach an unintended backend or a default/internal vhost:

```
GET / HTTP/1.1
Host: internal-admin.corp.local

GET / HTTP/1.1
Host: localhost
```

Where the proxy forwards an unknown `Host` to a default backend, or selects the upstream from `Host`, this reaches staging, admin, or internal-only applications. Combine with [origin exposure](origin-exposure.md) (default vhost) and the nginx [variable proxy_pass](../nginx/variable-proxy-pass-ssrf.md) case where `Host` chooses the upstream.

## Exploitation

- Enumerate which forwarding headers the origin trusts by sending each against an IP-gated path or a rate-limited endpoint and watching the effect.
- Rotate `X-Forwarded-For` per request to beat per-IP throttling during brute force and stuffing.
- Try internal hostnames and `localhost` in `Host` to reach internal vhosts and admin backends.

## Tools

- Burp (Repeater/Intruder, header rotation); the Param Miner extension for header discovery.

## References

- PortSwigger Web Security Academy: Host header attacks; HTTP headers and access control
- nginx/Apache: real_ip / remote IP configuration
