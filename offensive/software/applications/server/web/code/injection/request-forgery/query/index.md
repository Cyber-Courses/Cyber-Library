---
title: "Query: parameter-level SSRF allowlist bypasses"
order: 3
description: "When the fetched URL is built from query parameters, redirects and parameter pollution defeat an allowlist after it has already approved the request."
keywords:
  - SSRF query parameter
  - open redirect SSRF
  - HTTP parameter pollution
  - allowlist bypass
  - url parameter
---

# Query

Many fetch sinks take their target from a query parameter: `?url=`, `?target=`, `?image=`, `?next=`. The validation usually runs once, on the string in that parameter, and then the request is issued. The two pages here break the assumption that passing validation once means the final request is safe.

- **[Redirect-based bypass](bypassing-using-a-redirect.md)**: the submitted URL points at a host the allowlist permits, but that host responds with a redirect to an internal target. A client that follows redirects validates the first URL and connects to the second.
- **[Parameter pollution](parameter-pollution.md)**: supplying the parameter twice (`?url=allowed&url=internal`) exploits a mismatch between the component that validates and the component that fetches, when they disagree on which occurrence wins.

Both live at the query layer because they manipulate how the parameter is parsed and followed, not how the host or path is written. They frequently combine with the [Authority](../authority/index.md) tricks: the final, post-redirect or post-pollution target is still an internal host written with one of those encodings.

## Tools

- **Burp Suite**: manipulating `?url=` style parameters and replaying the built request.
- **SSRFmap**: automating query-parameter payload delivery against the sink.
- **curl**: issuing duplicated and redirecting parameter values to observe the final target.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
