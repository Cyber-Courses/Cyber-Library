---
title: "nginx merge_slashes and normalization: location matching that bypasses rules"
description: "Exploiting nginx slash and path handling: merge_slashes off, encoded slashes, and prefix-location matching that disagrees with the filesystem or an upstream, bypassing location-based access rules."
keywords:
  - merge_slashes
  - nginx normalization
  - encoded slash
  - location matching
  - access control bypass
---

# merge_slashes and normalization

nginx decides which `location` handles a request from the normalized URI, then may map it to a file or forward it upstream. Its slash and encoding handling can be tuned in ways that let a crafted path dodge a `location`-based rule or reach a file the rule meant to protect.

## merge_slashes off

By default nginx collapses repeated slashes, but `merge_slashes off;` (set to preserve URLs for some apps) changes matching. A rule meant to guard `/admin` can be dodged with doubled slashes when matching and the upstream disagree:

```
GET //admin/ HTTP/1.1
GET /admin//../admin/ HTTP/1.1
```

With merging off, `//admin` may not match a `location = /admin` or a prefix rule the same way the backend resolves it, so an access rule scoped to the single-slash form is bypassed while the resource still resolves.

## Prefix-location pitfalls

- A prefix `location /api` (no regex, no trailing slash) matches `/api`, `/api2`, and `/api-internal`, which can expose more than intended.
- An `internal;` location protecting an `X-Accel-Redirect` target can be reached directly if another rule maps to the same path.
- Mixing regex and prefix locations with different `proxy_pass`/`root` leads to a request being matched by one block but served/forwarded by another.

## Encoded slashes and dot-segments

nginx normalizes the request URI before `location` matching: it percent-decodes `%XX`, resolves `.`/`..` dot-segments, and (unless `merge_slashes off`) collapses repeated slashes. So a plain encoded traversal does **not** survive on the nginx side for its own matching, and `%2e%2e%2f` is resolved like `../` before a rule is chosen.

The encoded-slash trick therefore belongs to a proxy [normalization mismatch](../reverse-proxy-and-edge/normalization-mismatch.md), not to nginx matching its own file paths: it works when nginx forwards to an **origin that decodes differently**. The relevant nginx detail is how `proxy_pass` passes the URI: with no URI part in `proxy_pass`, nginx forwards the normalized request URI; with a URI part, it forwards the matched-and-rewritten path. A mismatch between what nginx normalized and what the upstream re-decodes is where an encoded `..%2f` lands at the origin:

```
GET /public/..%2f..%2fadmin HTTP/1.1     # exploitable at the origin behind nginx, per the proxy page
```

## Exploitation

- Against a `location`-protected path (401/403 at `/admin`, `/internal`), the nginx-side wins are doubled slashes (where `merge_slashes off`) and trailing variants; a `200` on the protected resource confirms a matching gap.
- Enumerate prefix-match over-reach by probing sibling paths that share a protected prefix.
- Encoded slashes and dot-segments are normalized by nginx itself, so save those for when nginx proxies an origin: pair them with the [reverse-proxy normalization mismatch](../reverse-proxy-and-edge/normalization-mismatch.md) technique against the backend.

## Tools

- **curl --path-as-is**, Burp Intruder with slash/encoding payloads.

## References

- nginx: location, merge_slashes, internal
- PortSwigger Web Security Academy: Access control bypass
