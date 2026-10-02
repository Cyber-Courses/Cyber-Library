---
title: "HTTP request smuggling (desync)"
description: "Desynchronizing a front-end and back-end that measure request length differently, using Content-Length and Transfer-Encoding in the CL.TE, TE.CL, and TE.TE variants."
keywords:
  - HTTP request smuggling
  - desync
  - CL.TE
  - TE.CL
  - TE.TE
  - Transfer-Encoding
---

# Request smuggling

Request smuggling (desync) abuses a chain where a front-end proxy and a back-end server disagree on where one request ends and the next begins. HTTP/1.1 offers two ways to state a body's length, `Content-Length` (CL) and `Transfer-Encoding: chunked` (TE); if the two servers trust different ones, an attacker can hide the start of a second request inside the first. That smuggled prefix is then glued to the next client's request, enabling front-end control bypass, request capture, and cache poisoning.

**CL.TE**: the front-end uses `Content-Length`, the back-end uses `Transfer-Encoding`. The front forwards the whole body by byte count; the back stops at the chunked terminator `0\r\n\r\n`, leaving the trailing bytes as the beginning of the next request:

```
POST / HTTP/1.1
Host: victim
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

The back-end treats `0\r\n\r\n` as the end and leaves `G`, which prefixes the next request (turning it into `GPOST ...` or a crafted smuggled request).

**TE.CL**: the reverse, front-end honors `Transfer-Encoding`, back-end honors `Content-Length`. The chunk sizes are crafted so the back-end's `Content-Length` cuts the body early, smuggling the remainder.

**TE.TE**: both support `Transfer-Encoding`, but one is induced to ignore it with an obfuscated header one parser accepts and the other rejects (`Transfer-Encoding: xchunked`, a space before the colon, a duplicated header, or `Transfer-Encoding:\tchunked`). Whichever end falls back to `Content-Length` then desyncs like CL.TE or TE.CL.

The payloads are byte-sensitive (exact CR/LF, chunk sizes), and HTTP/2 downgrade smuggling extends the idea where a front-end rewrites h2 to h1. Confirm a desync with timing or a benign prefix before weaponizing, since a wrong length can break the shared connection.

## References

- PortSwigger Web Security Academy: HTTP request smuggling
- RFC 9112: HTTP/1.1 message length (Content-Length vs Transfer-Encoding)
