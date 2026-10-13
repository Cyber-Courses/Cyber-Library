---
title: "Authority: host and port targeting in SSRF"
order: 1
description: "The authority is the host and port a request lands on. Loopback and link-local targets, alternate IP encodings, hostname resolution tricks, and port reach all live here."
keywords:
  - SSRF authority
  - localhost
  - 169.254.169.254
  - IP encoding
  - port scanning
  - host allowlist bypass
---

# Authority

The authority component of a URL is the `host:port` that decides *where* the request lands. It is the part a validator most wants to constrain, because allowing an arbitrary host is what turns a fetch into SSRF, and it is also the part with the most bypasses, because a host can be written many equivalent ways and resolved at a time of the attacker's choosing.

Targeting the authority covers three moves:

- **Reaching internal hosts.** Loopback (`127.0.0.1`, `[::1]`), all-interfaces (`0.0.0.0`), the RFC1918 ranges, and the cloud link-local metadata address `169.254.169.254` are the high-value destinations. The pages below reach them even when the literal strings are blocked.
- **Writing the host so a filter misses it.** [IP address](ip-address.md) encodings (decimal, octal, hex, IPv6-mapped, shorthand) and hostname-layer tricks under [Domain name](domain-name/index.md) (DNS rebinding, internationalized domains) all resolve to an internal target while reading as something else.
- **Choosing the port.** [Port](port.md) covers sweeping and selecting ports, which turns a blind fetch into a scanner and aims later scheme payloads at the right service.

## The validator's weak point

Host allowlists and blocklists operate on a string, but the network connects to a resolved IP. Every page in this subtree widens the gap between the two: the string the validator inspects and the address the socket actually reaches. A check that compares the literal host, normalizes differently from the HTTP client, or resolves the name once for validation and again for the connection is defeated by one of the techniques here.

## Tools

- **SSRFmap**: automated SSRF exploitation across common internal targets and authority bypasses.
- **Burp Collaborator**: detecting blind SSRF reach via out-of-band DNS and HTTP callbacks.
- **curl**: manual probing of hosts, ports, and schemes to confirm which authority forms connect.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
