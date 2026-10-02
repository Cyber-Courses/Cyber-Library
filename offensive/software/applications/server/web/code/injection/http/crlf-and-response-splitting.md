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

Injecting a double CRLF ends the header section and writes attacker-controlled bytes into the current response's body, which yields cross-site scripting when that body is rendered:

```
/redirect?url=%0d%0a%0d%0a<script>alert(document.domain)</script>
```

This is response-body injection: the double CRLF only starts the body of the one response, it does not by itself emit a second status line. True response splitting, a separately framed second response that can be cached as its own entry, additionally requires control over the first response's framing (for example a reflected `Content-Length` that ends the first response so the injected bytes are parsed as a new one), which modern servers and HTTP libraries largely prevent. So the dependable consequences here are header injection (setting cookies, security, or CORS headers) and XSS via the injected body; full split-response cache poisoning needs the extra framing and a permissive stack.

The major caveat is that most modern servers and HTTP libraries reject or strip CR and LF in header-setting APIs (Java's `HttpServletResponse`, Node, and current application servers), so response splitting largely survives on older stacks, custom header writers, or components that build raw response text. Where the framework blocks raw CRLF, the same CRLF primitive may still matter inside other protocols the app speaks (for example SMTP or log lines), covered under their own topics.

## References

- PortSwigger Web Security Academy: HTTP response header injection
- OWASP Testing Guide: Testing for HTTP Splitting/Smuggling
