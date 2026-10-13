---
title: "HTTP method and request-line abuse"
order: 3
description: "Abusing the HTTP request line: method override headers and verb tampering to bypass method-based access control, and request-line injection."
keywords:
  - method override
  - X-HTTP-Method-Override
  - verb tampering
  - request line injection
  - HTTP method
---

# Request line

The request line (`METHOD URI VERSION`) determines how a request is routed and authorized, and applications that make access decisions on the method can be subverted by changing which method is applied.

Method-override mechanisms let a client change the effective method after the front-end has seen the original. Many frameworks honor `X-HTTP-Method-Override`, `X-HTTP-Method`, `X-Method-Override`, or a `_method` body/query field, converting a `POST` into `PUT`/`DELETE`/`PATCH`. Where a proxy or WAF enforces controls per method (blocking `DELETE` but allowing `POST`), the override reaches the back end as the privileged method and bypasses that control:

```
POST /api/users/5 HTTP/1.1
X-HTTP-Method-Override: DELETE
```

Verb tampering abuses incomplete method handling. If an access rule is written for specific verbs (`GET` and `POST`), an uncommon method such as `HEAD`, `PUT`, or an arbitrary token can slip past the rule while the application still processes the request, a classic misconfiguration in container and framework authorization filters.

Request-line injection proper arises when an application builds an outbound request line from input (an HTTP client or proxy constructing `GET <input> HTTP/1.1`). Because the method token precedes the input, unescaped spaces or CR/LF cannot change the method of that request, but they can alter the target path or version, append headers, or, with complete request framing, inject a separate later request whose method does differ. This overlaps with server-side request forgery when the influenced request reaches an internal service, so it is often chained there.

The test is to resend a request with each override header and each unusual verb and watch for a changed authorization or routing outcome.

## Tools

- **Burp Repeater**: swap methods, add override headers, and watch authorization and routing changes.
- **Turbo Intruder**: iterate across override headers and unusual verbs at scale.
- **curl**: send arbitrary methods and method-override headers from the command line.

## References

- OWASP Testing Guide: Testing for HTTP Verb Tampering
- RFC 9112: HTTP/1.1 request line
