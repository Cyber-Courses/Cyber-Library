---
title: "HTTP request smuggling and desynchronization: CL.TE, TE.CL, and front-back disagreements"
description: Exploiting inbound HTTP framing where a reverse proxy and origin server disagree on request boundaries—CL.TE, TE.CL, and TE.TE desync, confirmation probes, and weaponization into request hijacking and cache poisoning, for lab reproduction only.
keywords:
  - HTTP request smuggling
  - desync
  - CL.TE
  - TE.CL
  - TE.TE
  - cache poisoning
---

# Request smuggling

**HTTP request smuggling** exploits two HTTP agents in line—typically a **front end** (CDN, load balancer, reverse proxy, or WAF) and a **back end** origin server—that disagree about where one request ends and the next begins. HTTP/1.1 keeps connections alive and pipelines requests back-to-back on the same socket, so the boundary between messages is purely a matter of byte counting. If the front end thinks a request ends at byte *N* while the back end thinks it ends at byte *M*, the bytes in between are read by the back end as the **start of the next request**. That attacker-supplied prefix is prepended to whichever request arrives next on that connection—often a different user's.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Smuggling corrupts requests belonging to other users of the shared connection, so reproduce it only in an isolated lab with pinned component versions. Testing third-party infrastructure without written authorization is unlawful.

## Overview

The body of an HTTP/1.1 request is delimited one of two ways:

- **`Content-Length`** — an exact byte count of the body.
- **`Transfer-Encoding: chunked`** — a series of hex-prefixed chunks terminated by a `0` chunk and a trailing `\r\n\r\n`.

The specification says a message must not carry both, and that `Transfer-Encoding` wins if it does. Real deployments routinely receive both anyway, and the two servers in the chain do not always resolve the conflict identically. Every smuggling primitive is a way to make the front end and back end pick **different** delimiters for the same bytes.

## The desync classes

| Class | Front end uses | Back end uses | Mechanism |
|-------|---------------|---------------|-----------|
| **CL.TE** | `Content-Length` | `Transfer-Encoding` | Front end forwards the whole body by length; back end stops at the `0` chunk, leaving the remainder as a new request |
| **TE.CL** | `Transfer-Encoding` | `Content-Length` | Front end forwards by chunks; back end reads only `Content-Length` bytes, leaving trailing chunk data as a new request |
| **TE.TE** | `Transfer-Encoding` (obfuscated) | `Transfer-Encoding` (one side ignores it) | A malformed `Transfer-Encoding` header is honored by one hop and rejected by the other, collapsing to CL.TE or TE.CL |

TE.TE is reached by **obfuscating** the `Transfer-Encoding` header so that exactly one server stops recognizing it:

```
Transfer-Encoding: chunked
Transfer-Encoding: x
Transfer-Encoding:[tab]chunked
Transfer-Encoding: chunked[space]
X: X[\n]Transfer-Encoding: chunked
```

## Confirming a desync

Blind confirmation uses a **timing** probe that is safe because it only delays the attacker's own socket.

**CL.TE detection** — the front end forwards all `Content-Length` bytes; the back end sees a chunked body that ends at `0`, then waits for the rest of a request that never comes, producing a delay:

```
POST / HTTP/1.1
Host: lab.example
Content-Length: 4
Transfer-Encoding: chunked

1
A
X
```

**TE.CL detection** — the reverse: the back end reads a short `Content-Length` and hangs waiting for more chunk data.

```
POST / HTTP/1.1
Host: lab.example
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

A clear time delta between the obfuscated request and a baseline confirms which parser honors which header. Raw-socket tooling matters here: libraries that "helpfully" fix `Content-Length` or reorder headers destroy the payload, so send exact bytes and capture the origin socket to see what actually arrived.

## Weaponization

Once a connection desyncs, the smuggled prefix is a **partial request** the back end glues onto the next victim request. Common escalations:

### Bypassing front-end access controls

Front ends often enforce path-based authorization (`/admin` blocked from outside). Smuggle a prefix whose request line targets the restricted path; the back end, which trusts the front end, serves it:

```
POST / HTTP/1.1
Host: lab.example
Content-Length: 54
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: lab.example
X: X
```

### Capturing another user's request

Make the smuggled prefix an incomplete request whose body length is large, so the victim's following request is **appended into a parameter** and stored/reflected where the attacker can read it:

```
POST /comment HTTP/1.1
...
Content-Length: 400
Transfer-Encoding: chunked

0

POST /comment HTTP/1.1
Host: lab.example
Content-Length: 400
Cookie: session=...
comment=
```

The next user's full request—headers, session cookie, CSRF token—lands in the `comment` field and is echoed back when the attacker views it.

### Web cache poisoning and response queue desync

A smuggled request can cause the back end to emit a response that the front end pairs with the **wrong** request, shifting the entire response queue by one. Every subsequent user on that connection receives the response intended for the previous request, and poisoned objects can be stored by a shared cache and served to all visitors.

## HTTP/2 downgrade and request tunnelling

When a front end speaks HTTP/2 to clients but **downgrades** to HTTP/1.1 toward the origin, the explicit length framing of HTTP/2 can be rewritten into ambiguous HTTP/1.1, reintroducing CL.TE/TE.CL even where the edge looked immune. **H2.CL** and **H2.TE** variants smuggle via an HTTP/2 body or injected `\r\n` in header values that the downgrade copies verbatim. Request tunnelling exploits the same downgrade to bind a smuggled request to the attacker's own connection, useful where connection reuse across users is disabled.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** with the **HTTP Request Smuggler** extension automates CL.TE/TE.CL/TE.TE probing and connection-state checks.
- **[Burp Repeater](https://portswigger.net/burp)** with "Content-Length" auto-update disabled, for hand-crafted raw requests over a single connection.
- **[smuggler](https://github.com/defparam/smuggler)** and **[h2csmuggler](https://github.com/BishopFox/h2csmuggler)** for scripted desync and HTTP/2 downgrade testing in a lab.

## References

- [PortSwigger Web Security Academy: HTTP request smuggling](https://portswigger.net/web-security/request-smuggling)
- [PortSwigger Research: HTTP/2 smuggling and request tunnelling](https://portswigger.net/research/http2)
- [CWE-444: Inconsistent Interpretation of HTTP Requests](https://cwe.mitre.org/data/definitions/444.html)
- [RFC 7230: HTTP/1.1 Message Syntax and Routing](https://datatracker.ietf.org/doc/html/rfc7230)
