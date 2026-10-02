---
title: "HTTP injection and desynchronization: smuggling, header abuse, and message boundary confusion"
description: How inbound HTTP messages are parsed and framed, and how disagreements between a front-end proxy and an origin server over request boundaries, or newline acceptance in header values, become exploitable injection primitives.
keywords:
  - HTTP request smuggling
  - desync
  - HTTP response splitting
  - CRLF injection
  - header injection
  - cache poisoning
---

# HTTP injection

**HTTP injection** is the class of attacks in which an attacker manipulates the *structure* of an HTTP message, its request line, headers, body, and the boundaries between them, rather than the data a single endpoint processes. HTTP is a text protocol whose framing is defined by exact byte sequences: a blank line (`\r\n\r\n`) ends the headers, `Content-Length` and `Transfer-Encoding` delimit the body, and a single `\r\n` separates one header from the next. When an attacker controls those bytes, or when two servers in a chain disagree about where a message ends, the result is request smuggling, response splitting, and header injection.

## Overview

Modern deployments chain several HTTP agents: a CDN, a load balancer, a reverse proxy, a WAF, and finally the application server. Each one re-parses the byte stream, and the specification leaves enough ambiguity (duplicate headers, obsolete line folding, conflicting length indicators) that two implementations can legitimately disagree. Two broad primitives follow:

1. **Framing disagreement between hops.** When a front end and a back end compute the end of a request differently, bytes the front end treats as body are read by the back end as the start of a *new* request. That smuggled prefix is glued onto the next connection, letting an attacker influence someone else's request, bypassing front-end access controls, poisoning caches, or capturing victim data. See [Request smuggling](request-smuggling.md).

2. **Newline injection into a single message.** When application code builds a response header (or an app-generated forward request) by concatenating user input, an unescaped `\r\n` lets the attacker end the current header early and inject their own headers or an entire second message. See [HTTP response splitting](http-response-splitting.md) and [CRLF in header fields](crlf-in-header-fields.md).

The unifying observation is that the control plane (framing, header boundaries) and the data plane (values) share the same byte stream, and any attacker-controlled byte that the parser reads as structure crosses from data into control.

## Why hops disagree

- **Specification latitude.** RFC 7230 forbids sending both `Content-Length` and `Transfer-Encoding`, but implementations still receive such requests and must pick one, and they do not all pick the same one.
- **Normalization order.** A proxy may rewrite header names, strip whitespace, or downcase tokens before forwarding; the origin sees a subtly different message than the one the proxy validated.
- **Obsolete features.** Line folding, chunk extensions, and unusual `Transfer-Encoding` spellings (`Transfer-Encoding:\tchunked`, duplicated headers) are handled inconsistently across the chain.
- **Reflected newlines.** Values copied into `Location`, `Set-Cookie`, or custom headers without CR/LF stripping carry the attacker's framing straight into the response.

## Impact

Smuggling yields request hijacking (capturing another user's full request, including credentials), front-end security-control bypass, and web cache poisoning that serves attacker content to every subsequent visitor. Response splitting and CRLF injection yield reflected header injection, cookie fixation, open redirect with injected headers, and cache poisoning. The common denominator is that a single crafted request affects messages beyond the attacker's own session, which is what makes these bugs high-impact and why they must be contained to a lab.

## Pages

| Page | Focus |
|------|--------|
| [Request smuggling](request-smuggling.md) | CL.TE, TE.CL, and TE.TE framing disagreements between a front end and an origin, and how a smuggled prefix is weaponized (lab-only) |
| [HTTP response splitting](http-response-splitting.md) | CRLF reflected into response headers to terminate headers early, inject new ones, and poison caches |
| [CRLF in header fields](crlf-in-header-fields.md) | Newline acceptance in header values built from user input, across response and app-generated request headers |

## References

- [PortSwigger Web Security Academy: HTTP request smuggling](https://portswigger.net/web-security/request-smuggling)
- [OWASP: HTTP Response Splitting](https://owasp.org/www-community/attacks/HTTP_Response_Splitting)
- [CWE-444: Inconsistent Interpretation of HTTP Requests (HTTP Request Smuggling)](https://cwe.mitre.org/data/definitions/444.html)
- [CWE-113: Improper Neutralization of CRLF Sequences in HTTP Headers](https://cwe.mitre.org/data/definitions/113.html)
- [RFC 7230: HTTP/1.1 Message Syntax and Routing](https://datatracker.ietf.org/doc/html/rfc7230)
