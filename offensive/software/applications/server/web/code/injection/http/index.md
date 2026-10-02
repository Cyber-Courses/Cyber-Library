---
title: "HTTP injection: headers, CRLF, smuggling, and parameter pollution"
description: "Exploiting the HTTP message itself: header-value abuse, CRLF response splitting, request smuggling when front and back ends disagree, and HTTP parameter pollution."
keywords:
  - HTTP injection
  - CRLF injection
  - request smuggling
  - host header injection
  - HTTP parameter pollution
---

# HTTP

HTTP injection treats the HTTP message as the dangerous primitive: when an application or an intermediary builds or parses requests and responses from untrusted input, the structure of the message itself can be subverted. This is distinct from injecting into a query or command carried inside the request; here the target is the headers, the line structure, the body framing, and the way a chain of proxies and servers agrees (or disagrees) on where one message ends.

The attacks fall into a few families. Header-value abuse trusts attacker-set headers such as `Host` and `X-Forwarded-*` for routing, access control, or link generation. CRLF injection splits a response when a carriage-return/line-feed reaches a response header. Request smuggling desynchronizes a front-end and back-end that measure request length differently. Parameter pollution supplies duplicate parameters that components resolve inconsistently.

A recurring theme is that modern servers and frameworks have hardened many of these (stripping CR/LF from header APIs, normalizing `Content-Length`/`Transfer-Encoding`), so each technique's reach depends on the exact stack, and fingerprinting the front-end and back-end is part of the work.

## Techniques

- **[Header injection](header-injection.md)**: abuse `Host` and forwarding/override headers for poisoning and access bypass.
- **[CRLF and response splitting](crlf-and-response-splitting.md)**: inject CR/LF into a response header to add headers or split the response.
- **[Request smuggling](request-smuggling.md)**: desynchronize front-end and back-end with `CL.TE`, `TE.CL`, and `TE.TE`.
- **[Parameter pollution](parameter-pollution.md)**: duplicate parameters that components resolve differently.
- **[Request line](request-line.md)**: method override and request-line abuse.
- **[Request body](request-body.md)**: content-type and body-parsing confusion.

## References

- RFC 9110/9112 (HTTP semantics and HTTP/1.1 messaging)
- PortSwigger Web Security Academy: HTTP request smuggling, Host header attacks
