---
title: "CRLF injection and HTTP response splitting"
description: "Injecting carriage-return/line-feed into a reflected response header to add headers, set cookies, or split the response into a second attacker-controlled body."
keywords:
  - CRLF injection
  - response splitting
  - header injection
  - Set-Cookie injection
  - %0d%0a
---

# CRLF and response splitting

HTTP headers are separated by CR/LF (`\r\n`, `%0d%0a`), and a blank line (`\r\n\r\n`) separates headers from the body. When untrusted input is placed into a response header without stripping those bytes, an attacker can inject new header lines, and with a double CRLF, a whole second response.

The input is usually a value the server reflects into a header, such as a redirect target copied into `Location`, or a parameter echoed into `Set-Cookie` or a custom header. Injecting one CRLF adds a header:

```
/redirect?url=/home%0d%0aSet-Cookie:%20sid=attacker
-> Location: /home
   Set-Cookie: sid=attacker
```

Injecting a double CRLF ends the headers and writes a body, splitting the response so the attacker controls content (stored or reflected XSS, or a poisoned cache entry):

```
/redirect?url=%0d%0a%0d%0a<script>alert(document.domain)</script>
```

The consequences are header injection (setting cookies, security headers, or CORS headers), cross-site scripting via the injected body, and web cache poisoning when the split response is cached.

The major caveat is that most modern servers and HTTP libraries reject or strip CR and LF in header-setting APIs (Java's `HttpServletResponse`, Node, and current application servers), so response splitting largely survives on older stacks, custom header writers, or components that build raw response text. Where the framework blocks raw CRLF, the same CRLF primitive may still matter inside other protocols the app speaks (for example SMTP or log lines), covered under their own topics.

## References

- PortSwigger Web Security Academy: HTTP response header injection
- OWASP Testing Guide: Testing for HTTP Splitting/Smuggling
