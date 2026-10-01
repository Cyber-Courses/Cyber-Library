---
title: "HTTP parameter pollution for SSRF allowlist bypass"
description: "Supplying the target parameter twice exploits a disagreement between the component that validates the URL and the one that fetches it over which duplicate occurrence wins."
keywords:
  - HTTP parameter pollution
  - duplicate parameter
  - last parameter wins
  - SSRF allowlist bypass
  - url=allowed&url=internal
---

# Parameter pollution

When the fetch target comes from a query parameter, supplying that parameter twice can split the validator and the fetcher. If the code that checks the allowlist reads one occurrence and the code that issues the request reads the other, a single query carries both an allowed value (for the check) and an internal value (for the fetch).

## Last-wins versus first-wins

Platforms disagree on which duplicate of a repeated parameter a given API returns. Supplying both an allowed and an internal value exploits that disagreement:

```
?url=https://allowed.example&url=http://169.254.169.254/latest/meta-data/
```

If the validator reads the first `url` (allowed) and the HTTP layer reads the last `url` (internal), the request passes the check and fetches the metadata endpoint. The reverse ordering works against a first-wins fetcher paired with a last-wins validator:

```
?url=http://127.0.0.1/&url=https://allowed.example
```

The vulnerable condition is simply that the two stages resolve duplicates differently. Common splits: one stage uses a framework helper that returns the first value while another splits the raw query string and takes the last, or a proxy and an application disagree.

## Encoded and nested delimiters

Where a single parameter is parsed into sub-values, an encoded delimiter smuggles a second value past the check. A `;` or an encoded `&` inside the value can introduce a second parameter after validation:

```
?url=https://allowed.example%26url=http://127.0.0.1/
?url=https://allowed.example%3Burl=http://127.0.0.1/
```

The validator sees one opaque value; a later decode splits it into two, and the internal one is selected.

## Confirming the split

Send the duplicated parameter with one value pointing at an attacker-controlled listener and the other at an allowed host, and watch which one receives the connection. That identifies whether the fetcher takes first or last, which fixes the ordering for the real internal target. The selected internal value still uses the [Authority](../authority/index.md) encodings when its literal form is blocked, and pairs naturally with a [redirect](bypassing-using-a-redirect.md) when a single clean internal URL is needed.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP: Testing for HTTP Parameter Pollution](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution)
