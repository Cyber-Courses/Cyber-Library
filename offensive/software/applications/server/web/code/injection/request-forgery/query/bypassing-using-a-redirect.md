---
title: "Redirect-based SSRF allowlist bypass"
description: "A URL on an allowed host that responds with a redirect to an internal target defeats a validator that checks only the submitted URL, because the client follows the redirect to the real destination."
keywords:
  - redirect SSRF
  - open redirect
  - 302 to internal
  - Location header
  - allowlist bypass
  - follow redirects
---

# Redirect-based bypass

Validation usually runs on the URL the attacker submits. If that URL points at an allowed host but the host replies with a redirect to an internal address, a client that follows redirects validates the first URL and connects to the second. The check sees an external, permitted target; the socket reaches `127.0.0.1` or the metadata service.

## The two-hop structure

The attacker supplies a URL whose host passes the allowlist. That host, which the attacker controls or which has an open-redirect flaw, answers with a `3xx` and a `Location` pointing inside:

```
Submitted:  https://attacker.example/r
Response:   HTTP/1.1 302 Found
            Location: http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

The validator approves `attacker.example` (public, allowed). The client, following the redirect, issues the second request to the metadata endpoint and returns its body. Any internal target works as the `Location`:

```
Location: http://127.0.0.1:8080/admin
Location: http://10.0.0.5:6379/
```

## Open redirects on allowed hosts

When the attacker cannot host the first endpoint, an open redirect on an already-allowed host serves the same purpose. A permitted domain with a redirect parameter turns into the first hop:

```
https://allowed.example/redirect?next=http://127.0.0.1/
https://allowed.example/out?url=http%3A%2F%2F169.254.169.254%2F
```

The submitted URL is unquestionably on the allowlist, yet it bounces the client inward.

## Scheme changes across the hop

Some clients allow the scheme to change on a redirect, which upgrades a redirect bypass into a scheme attack: a `Location` of `file:///etc/passwd` or `gopher://127.0.0.1:6379/_...` makes an HTTP fetch pivot to a local read or a raw-socket write. Whether this works depends on the client's cross-scheme redirect policy, so test a `file://` or `gopher://` `Location` once an HTTP redirect is confirmed to be followed.

## Why it beats submit-time validation

The flaw is validating the request target once, at submission, and trusting every subsequent hop. Re-validating the destination of each redirect against the policy (or disabling redirect following on the sink) closes it. Until then, redirecting is the most reliable way past an allowlist that only inspects the URL the user typed. The final internal target still uses the [Authority](../authority/index.md) encodings when its literal form is filtered.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
