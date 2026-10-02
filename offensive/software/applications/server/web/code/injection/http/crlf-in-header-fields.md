---
title: "CRLF injection in HTTP header fields: newline abuse in request and response headers"
description: Exploiting carriage-return and line-feed bytes in header values built from user input, the shared primitive behind response splitting, cookie and log injection, and newline smuggling into application-generated outbound requests.
keywords:
  - CRLF injection
  - header injection
  - newline injection
  - log injection
  - cookie injection
  - SSRF header smuggling
---

# CRLF in header fields

**CRLF injection** is the root primitive behind HTTP header manipulation: a header value, which must be a single line, is built from user input that still contains a carriage return (`\r`, `%0d`) or line feed (`\n`, `%0a`). Because HTTP delimits headers with `\r\n` and ends the header block with `\r\n\r\n`, any attacker-controlled newline that survives into the stream is read as **structure**, not text. The same injection primitive surfaces in two directions, reflected into **response** headers the server sends to the client, and injected into **request** headers when application code builds its own outbound HTTP messages.

## Overview

Wherever user input flows into a header value without CR/LF removal, a newline lets the attacker end the intended token and append arbitrary header lines:

```
User-supplied "name" header value:
  Dave%0d%0aX-Injected: true
Emitted on the wire:
  X-Name: Dave
  X-Injected: true
```

The vulnerability is identical to [HTTP response splitting](http-response-splitting.md) at the byte level; this page treats the newline-acceptance primitive itself and the contexts beyond redirect reflection where it bites.

## The two directions

- **Outbound responses (common).** Values copied into `Set-Cookie`, `Location`, `Content-Disposition`, or custom `X-` headers. This is the response-splitting surface: injected headers, forged bodies, and cache poisoning.
- **App-generated requests (rarer, high value).** When server code constructs an HTTP request to an internal service and places user input into a **request** header (an API key header, a forwarded `Host`, a `X-Forwarded-For`), an injected `\r\n` can add request headers or a body the internal service honors, pairing CRLF injection with server-side request forgery to reach internal endpoints with attacker-chosen headers.

## Injection payloads

| Payload | Encoded | Use |
|---------|---------|-----|
| CRLF + header | `%0d%0aX-Injected:%201` | Append a single header |
| Bare LF + header | `%0aX-Injected:%201` | Works where servers accept `\n` alone as a separator |
| CRLF ×2 + body | `%0d%0a%0d%0a<payload>` | Terminate headers, inject a body |
| Encoded newline in JSON/UTF-8 | `\u000d\u000a`, `%E5%98%8A` | Survive filters that only check literal `\r`/`\n` |
| Tab/space obfuscation | `%0d%09`, `%0d%20` | Defeat naive equality checks on `\r\n` |

## Exploitation contexts

### Cookie injection and fixation

A value reflected into `Set-Cookie` lets the attacker set or overwrite cookies, fixing a known session identifier or planting attacker-controlled preference cookies:

```
/set?lang=en%0d%0aSet-Cookie:%20session%3Dattacker-fixed
```

### Log injection

Headers like `User-Agent` or `Referer`, and reflected values written to application logs, carry newlines straight into the log file. Injected lines can forge log entries, break log parsers, or, when logs are rendered in a web dashboard, deliver stored payloads to whoever reviews them:

```
User-Agent: Mozilla/5.0%0d%0a[CRITICAL] forged admin login from 10.0.0.9
```

### Smuggling headers into outbound requests

Where application code forwards a user-supplied value into an internal HTTP call, a CRLF injects additional request headers. Combined with an SSRF sink, this lets the attacker reach an internal service *and* control the headers it receives (authentication headers, routing hints), widening what the forged request can do.

## Finding the primitive

1. **Enumerate sinks.** Identify every parameter, path segment, and inbound header whose value reaches an outbound header, response or app-generated request.
2. **Test newline survival.** Submit `%0d%0a`, `%0a` alone, and encoded/obfuscated variants, then read the **raw** bytes of the response (or capture the outbound request on the internal socket) to see whether a second header line materializes.
3. **Characterize the context.** Determine whether you can append headers only, or also inject a blank line and a body; whether a bare LF is enough; and whether a downstream cache or internal service will act on the injected structure.
4. **Escalate** to cookie fixation, log forgery, response splitting, or request-header smuggling per the sink.

Because many clients and proxies silently strip or normalize CR/LF, always confirm against the unmodified byte stream rather than a rendered or re-serialized view.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** Repeater/Intruder for CR/LF fuzzing and raw-byte response inspection.
- **[crlfuzz](https://github.com/dwisiswant0/crlfuzz)** for scriptable discovery of CRLF-injectable parameters.
- **[OWASP ZAP](https://www.zaproxy.org/)** active scan rules for header/CRLF injection.

## References

- [CWE-93: Improper Neutralization of CRLF Sequences (CRLF Injection)](https://cwe.mitre.org/data/definitions/93.html)
- [CWE-113: Improper Neutralization of CRLF Sequences in HTTP Headers](https://cwe.mitre.org/data/definitions/113.html)
- [OWASP: HTTP Response Splitting](https://owasp.org/www-community/attacks/HTTP_Response_Splitting)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
