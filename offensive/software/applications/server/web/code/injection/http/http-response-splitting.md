---
title: "HTTP response splitting: CRLF injection into response headers and cache poisoning"
description: Exploiting user-controlled data reflected into Location, Set-Cookie, or custom response headers where injected CRLF terminates the header block early, appends attacker headers, forges a second response, and poisons shared caches.
keywords:
  - HTTP response splitting
  - CRLF injection
  - cache poisoning
  - header injection
  - cookie injection
  - open redirect
---

# HTTP response splitting

**HTTP response splitting** occurs when application code places user-controlled data into a **response header** without stripping carriage-return (`\r`, `%0d`) and line-feed (`\n`, `%0a`) bytes. Because a single `\r\n` separates one header from the next and a blank line (`\r\n\r\n`) ends the header block, an attacker who injects those bytes can terminate the intended header early, append headers of their own, and—by injecting a full `\r\n\r\n`—start an entirely new, attacker-controlled response body. The attacker is writing raw HTTP structure into a stream the developer assumed was a flat string.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Cache poisoning affects every user served the poisoned object, so validate impact only against infrastructure you control. Testing without written authorization is unlawful.

## Overview

A classic sink reflects a request parameter into a redirect:

```java
// lang comes from a query-string parameter
response.setHeader("Location", "/home?lang=" + lang);
```

The developer expects `lang=en`. The HTTP layer, however, writes whatever bytes `lang` contains directly into the header stream. Supplying:

```
en%0d%0aSet-Cookie:%20sessionid=attacker
```

produces two headers where one was intended:

```
Location: /home?lang=en
Set-Cookie: sessionid=attacker
```

The injection crosses from **data** into **protocol structure** because the `\r\n` the attacker supplied is interpreted as a header boundary rather than literal text.

## The injected bytes

| Sequence | Encoded | Effect |
|----------|---------|--------|
| Carriage return + line feed | `%0d%0a` | Ends the current header, begins a new one |
| Double CRLF | `%0d%0a%0d%0a` | Ends the header block; everything after is response body |
| Lone LF | `%0a` | Many servers accept a bare `\n` as a header separator |
| Unicode/overlong variants | `%E5%98%8A%E5%98%8D` | Some stacks decode these to `\r\n`, bypassing naive byte checks |

Reflection points worth probing are any header built from input: `Location` (redirects), `Set-Cookie` (language/region/tracking values), `Content-Disposition` (filenames), and custom `X-` headers populated from request data.

## Exploitation

### Injecting headers

The minimal primitive is appending a header the application never meant to send. Setting a cookie the attacker controls enables **session fixation**:

```
/redirect?url=x%0d%0aSet-Cookie:%20session=FIXED%3B%20Path=/
```

Overwriting security-relevant response headers (for example neutering a restrictive `Content-Security-Policy` by appending a permissive one, where the browser honors the injected value) widens the follow-on attack surface for script injection.

### Splitting a full second response

Injecting a double CRLF closes the header block and lets the attacker supply a complete body, including a status line for a forged response:

```
/page?p=x%0d%0a%0d%0a%3Chtml%3E%3Cscript%3Ealert(document.domain)%3C/script%3E%3C/html%3E
```

When the response is reflected into a context a browser renders, the injected body executes as same-origin script—response splitting escalating into reflected XSS through the header channel.

### Poisoning a shared cache

The highest-impact outcome pairs splitting with a caching layer. By controlling the split so that a benign-looking URL maps to an attacker-authored body, the response is stored by a CDN or reverse-proxy cache and served to **every subsequent visitor** of that URL. The attacker sends one request; the cache distributes the payload. The key is choosing a cacheable path and crafting the injected headers (`Cache-Control`, `Content-Length`) so the intermediary stores the forged response.

## Finding injectable reflections

1. **Map reflections into headers.** Fuzz each parameter and watch which ones surface in `Location`, `Set-Cookie`, or other response headers rather than the body.
2. **Probe CR/LF survival.** Send `%0d%0a`, `%0a` alone, and encoded/overlong variants; inspect the raw response bytes (not a rendered view) to see whether a new header line actually appears.
3. **Confirm the primitive.** Once a bare `\n` or full `\r\n` reaches the header stream, append a marker header and verify it is emitted as its own line.
4. **Escalate** to cookie injection, second-response forgery, or cache poisoning depending on what the reflected header and the downstream cache allow.

Raw-byte inspection matters: a proxy or client library that normalizes the response will hide a working split, so capture the unmodified stream.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** Repeater and Intruder for fuzzing parameters with CR/LF payloads and reading raw response bytes.
- **[OWASP ZAP](https://www.zaproxy.org/)** active scanner includes CRLF/response-splitting checks.
- **[crlfuzz](https://github.com/dwisiswant0/crlfuzz)** for fast, scriptable CRLF-injection discovery across many endpoints in a lab.

## References

- [OWASP: HTTP Response Splitting](https://owasp.org/www-community/attacks/HTTP_Response_Splitting)
- [CWE-113: Improper Neutralization of CRLF Sequences in HTTP Headers](https://cwe.mitre.org/data/definitions/113.html)
- [PortSwigger Web Security Academy: Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
