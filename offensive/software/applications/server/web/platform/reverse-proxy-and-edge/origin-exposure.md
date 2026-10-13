---
title: "Origin exposure: reaching the backend directly past the proxy, WAF, or CDN"
order: 2
description: "Finding and hitting the origin server behind a reverse proxy, WAF, or CDN to bypass edge access control, rate limiting, and filtering: origin IP discovery, default-vhost confirmation, and direct-to-origin requests."
keywords:
  - origin exposure
  - waf bypass
  - cdn bypass
  - origin ip
  - default vhost
---

# Origin exposure

When security is enforced only at the edge (a WAF, CDN, or reverse proxy), reaching the **origin directly** bypasses all of it: filtering, rate limits, geo/IP rules, and access controls that live only in the front tier. The attack is to discover the origin's address and send requests straight to it, with the public hostname in the `Host` header so the app still routes correctly.

## Finding the origin

- **DNS history and subdomains**: current and historical A records (certificate transparency via `crt.sh`, passive DNS, SecurityTrails) often reveal the pre-CDN IP or a `origin.`/`direct.`/`staging.` host that resolves straight to the backend.
- **Leaked IP**: email headers from the app (password-reset mail `Received:` chain), verbose error pages, `server-status`, SSRF responses, and other services on the same host.
- **Certificate and favicon pivots**: scan the IP space (Shodan/Censys) for the app's TLS certificate or favicon hash to find the origin among unrelated hosts.
- **Default vhost confirmation**: an IP that serves the app on its **default virtual host** (no/!matching `Host`) confirms origin. A default vhost also sometimes serves a *different*, less-protected app or a status page, which is itself a finding.

## Hitting it directly

Send the request to the origin IP while preserving the public `Host` so virtual-host routing and the application behave normally:

```bash
curl -k https://ORIGIN_IP/admin -H 'Host: app.example.com'
# or pin the hostname to the IP:
curl -k --resolve app.example.com:443:ORIGIN_IP https://app.example.com/admin
```

If the response matches the real site, the origin is reachable and the edge is bypassed. Try an empty or wrong `Host` too, to see what the default vhost serves.

## Exploitation

- Replay payloads the WAF blocked (SQLi, XSS, traversal) directly at the origin, which has no filtering.
- Defeat edge rate limiting and bot rules for credential stuffing and brute force by targeting the origin.
- Reach admin or internal paths that were only ACL'd at the edge, overlapping with [normalization mismatch](normalization-mismatch.md) (same goal, different route).

## Tools

- **crt.sh**, **SecurityTrails**, **Shodan/Censys**, CloudFlair-style origin-finders; **curl --resolve**.

## References

- PortSwigger Web Security Academy: Web application firewalls (bypassing)
- OWASP WSTG: Testing for network/infrastructure configuration
