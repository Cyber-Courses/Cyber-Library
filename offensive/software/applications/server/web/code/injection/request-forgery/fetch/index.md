---
title: "Fetch: the client that performs the SSRF request"
order: 5
description: "Orthogonal to URL grammar, the client that issues the outbound request decides what a malicious URL can do. A programmatic HTTP library and a headless browser behave very differently for the same URL."
keywords:
  - SSRF client
  - headless browser SSRF
  - programmatic HTTP client
  - redirect following
  - cookie reuse
---

# Fetch

The [Authority](../authority/index.md), [Path](../path.md), [Query](../query/index.md), and [Scheme](../scheme/index.md) subtrees describe *what URL* to send. Fetch is the other axis: *what performs the request*. The same malicious URL produces different results depending on the client, so the client is worth classifying before choosing a payload.

- **[Programmatic](programmatic.md)**: a library HTTP client such as `fetch`, `axios`, `got`, Java `HttpClient`, Python `requests`/`httpx`, or Go `net/http`. It sends the request with no JavaScript execution context and usually no interactive session, so the attack surface is the URL, the redirect policy, and whatever headers the server attaches.
- **[Headless browser](headless-browser.md)**: a full browser engine driven server-side (Puppeteer, Playwright, Selenium) for rendering, screenshots, or PDF export. It carries cookies and local storage, runs script, and follows subresource loads, so a single navigation can reuse an authenticated session, execute attacker HTML, and chain requests.

## Why the client matters

A programmatic client that refuses redirects and rejects non-`http` schemes is a narrow target. A headless browser pointed at the same URL is far wider: it renders attacker markup (so injected HTML and script run in a privileged context), it loads `file://` and internal subresources, and it may attach a real session cookie to the request. Deciding which one the application uses tells you whether to invest in raw-byte scheme payloads (programmatic) or in session reuse and rendered-script exfiltration (headless). Host-level tricks such as [DNS rebinding](../authority/domain-name/dns-rebinding.md) apply to both, because both resolve names.

## Tools

- **SSRFmap**: driving payloads once the client type is known.
- **Burp Suite**: fingerprinting client behavior (redirects, user agent, script execution) per request.
- **Burp Collaborator**: distinguishing clients by observing which out-of-band loads fire.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
