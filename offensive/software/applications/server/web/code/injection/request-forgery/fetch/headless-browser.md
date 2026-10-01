---
title: "Headless browser SSRF and session reuse"
description: "A server-driven headless browser fetches URLs with cookies, storage, and script execution, so an attacker-chosen navigation reaches internal hosts, reuses authenticated sessions, and runs injected markup in a privileged context."
keywords:
  - headless browser SSRF
  - Puppeteer
  - Playwright
  - screenshot service
  - PDF renderer
  - session reuse
  - metadata endpoint
---

# Headless browser

Screenshot services, PDF exporters, link unfurlers, and scraping back ends often drive a real browser engine (Puppeteer, Playwright, Selenium) from the server. When attacker input chooses the navigation target, that browser is an HTTP client that also runs JavaScript, carries cookies and storage, and loads subresources, so it is a far wider SSRF primitive than a plain library fetch.

## Reaching internal targets

A browser navigation reaches the same internal hosts as any SSRF, including loopback admin panels and the cloud metadata endpoint. Because the result is usually rendered (into a screenshot or PDF), the response comes straight back:

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://127.0.0.1:8080/admin
```

Rendering embeds the fetched content in the output image or document, so even a service that returns plain text or JSON is captured as pixels, sidestepping content-type handling.

## Local files and injected script

A headless engine often honors `file://`, and it executes markup. If the service renders attacker-supplied HTML (a common pattern in HTML-to-PDF features), injected script runs in the page context and can read a local or internal resource and place it in the output:

```html
<iframe src="file:///etc/passwd"></iframe>
<script>
fetch('http://127.0.0.1:2375/containers/json')
  .then(r => r.text()).then(t => { document.title = t; });
</script>
```

Whether a cross-origin `fetch` can read the response back depends on the engine's security settings; an `<iframe>` or `<img>` that simply renders the target into the page does not need a cross-origin read and works even when script reads are blocked.

## Session reuse is the escalation

The dangerous property unique to this client is the **session**. A rendering worker that reuses one browser profile, or that is handed a user's cookies to capture an authenticated view, carries those cookies. Cookie scoping still applies: the browser sends a stored cookie only to a URL that matches its `Domain` and `Path`, so this escalates when the profile holds a cookie valid for the target. That is common in practice, a session cookie scoped to a parent corporate domain (`.corp`) or an SSO cookie covers many internal hosts under it, so an attacker-chosen URL within that scope rides the credential:

```
http://internal-dashboard.corp/api/users/export
```

Where the profile holds a cookie for `internal-dashboard.corp` (or a parent domain that includes it), the browser attaches it and the rendered response returns privileged data. The boundary this abuses is reusing an authenticated profile whose cookies are scoped to reachable internal hosts while letting the navigation URL be attacker-influenced; a worker whose cookies are scoped only to the intended external target does not leak them inward.

## Confirming the client

Behavior that reveals a headless browser rather than a [programmatic](programmatic.md) client: execution of injected `<script>`, loading of subresources and redirects like a browser, a recognizable headless user agent, and rendering of non-HTML responses as images. Once confirmed, invest in session reuse and rendered-script exfiltration rather than raw-scheme payloads. Host tricks such as [DNS rebinding](../authority/domain-name/dns-rebinding.md) apply here too, and bridge the browser's origin boundary as well as the server's.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
