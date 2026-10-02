---
title: "Domain name tricks in SSRF host validation"
description: "When the target is written as a hostname rather than a literal IP, resolution timing and Unicode handling open gaps between the name a validator checks and the address the client connects to."
keywords:
  - SSRF hostname bypass
  - DNS rebinding
  - internationalized domain
  - punycode
  - host allowlist
---

# Domain name

When the SSRF target is supplied as a hostname rather than a literal IP, a second layer appears between the string and the socket: name resolution. A validator inspects the hostname as text, but the HTTP client hands that name to a resolver and connects to whatever address comes back, possibly at a different moment and possibly after Unicode processing has rewritten it. The two pages here exploit that layer.

- **[DNS rebinding](dns-rebinding.md)** attacks the *timing* of resolution. A name the attacker controls answers with a public address when the validator resolves it, then with `127.0.0.1` or a metadata IP when the client connects, so one hostname passes the check and hits an internal service.
- **[Internationalized domain](internationalized-domain.md)** attacks the *text* of the hostname. Unicode labels, punycode, and normalization let a name read as an allowed host to one comparison and resolve as an attacker host to another.

Both widen the same gap the whole [Authority](../index.md) subtree targets, between the host a filter sees and the host the request reaches, but they do it at the naming layer rather than through raw IP encodings.

## Tools

- **Singularity of Origin**: DNS rebinding attack framework for serving attacker-controlled resolution.
- **Burp Collaborator**: detecting when the server resolves and reaches an out-of-band hostname.
- **curl**: confirming which supplied hostname form the client resolves and connects to.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
