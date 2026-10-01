---
title: "Programmatic HTTP client SSRF"
description: "Library HTTP clients fetch an attacker-influenced URL with no browser engine. Redirect following, scheme handling, and header defaults decide how far a user-controlled URL reaches."
keywords:
  - programmatic SSRF
  - axios
  - requests
  - HttpClient
  - redirect following
  - user-controlled URL
  - outbound request
---

# Programmatic

Most SSRF sinks are a library HTTP call: `fetch`, `axios`, `got`, Java `HttpClient`, Python `requests`/`httpx`, Go `net/http`, Ruby `Net::HTTP`. These send the request without a JavaScript engine and usually without an interactive session, so the attack surface is the URL, the client's redirect and scheme policy, and whatever headers the code attaches.

## The core sink

The vulnerable shape is a user value flowing into the request target:

```js
const r = await axios.get(req.query.url);   // url is attacker-controlled
```

```python
r = requests.get(user_url, timeout=5)
```

Pointed at an internal address, the client fetches it and often returns the body or an error that reflects it, giving the attacker a window into the internal network.

## Redirect following widens reach

Library clients differ in how they follow redirects, and that behavior is a bypass surface. A URL on an allowed host that returns `Location: http://169.254.169.254/...` is followed by a client that validated only the first URL. Whether redirects are followed by default, how many hops are allowed, and whether the scheme may change across a hop all vary by library:

- `requests` follows redirects by default; `axios` follows them; Go `net/http` follows by default with a customizable policy.
- Some clients downgrade or refuse a scheme change (for example `https` to `file`) on redirect; others do not.

This is the basis of the [redirect-based bypass](../query/bypassing-using-a-redirect.md): validate the first hop, connect on a later one.

## Scheme handling

A programmatic client exposes whatever schemes its runtime registers. A Java client built on `URL` may honor `file`, `ftp`, `jar`, and `netdoc`; `curl`-backed clients honor `dict`, `gopher`, `ftp`, `tftp`, and more. The [Scheme](../scheme/index.md) subtree depends entirely on what the specific client accepts, so enumerate the registered handlers before committing to a scheme payload.

## What it cannot do

Unlike a [headless browser](headless-browser.md), a programmatic client does not execute returned HTML or script, and it does not carry an interactive session unless the code explicitly attaches credentials. So there is no rendered-script exfiltration, but also no automatic cookie reuse. The payoffs are reading internal responses, scanning via [Port](../authority/port.md) timing, and raw-byte interaction through a capable [Scheme](../scheme/index.md). Host tricks like [DNS rebinding](../authority/domain-name/dns-rebinding.md) apply because the client re-resolves names.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
