---
title: "SSRF URL path: encoding, semicolon parameters, and parser-normalization bypasses"
description: Path component of attacker-influenced URLs in server-side fetches—encoded slashes, path parameters, and differences between what the filter sees and what the HTTP client requests.
keywords:
  - SSRF
  - URL path
  - encoded slash
---

# Path (SSRF)

The **path** facet is everything after the authority up to `?` or `#`. Filters that only inspect host and scheme miss path tricks: `%2f` vs `/`, `;param=value` segments (older stacks), double-encoding, and Unicode normalization that changes the final bytes on the wire.

## Topics to test (in scope)

- Compare **allowlist** matching on **raw** input vs **normalized** URL the client library builds before connect.
- Path joining when the application **prefixes** a **base** URL with a user fragment—`..`, absolute-path references, and encoded separators.

## Pages

| Page | Focus |
|------|--------|
| [Encoded slashes](encoded-slash-and-normalization.md) | `%2F`, double encoding, normalization |

## See also

- [Request forgery (parent)](../index.md)
- [Authority](../authority/index.md)
- [Query](../query/index.md)
