---
title: "Path traversal and parser confusion in SSRF URLs"
description: "When the host is fixed but the path is user-controlled, traversal reaches other endpoints on the internal host, and authority/path parser differentials move the effective target past a prefix allowlist."
keywords:
  - SSRF path traversal
  - URL parser confusion
  - prefix allowlist bypass
  - backslash authority
  - at-sign host
  - fragment truncation
---

# Path

When a fetch sink fixes the host and lets the attacker influence only the path, two avenues open: traversal that escapes a constrained base path to reach other endpoints on the same internal host, and parser differentials where characters in the path position change which authority the request actually targets.

## Escaping a fixed base path

A sink that builds `http://internal-api/v1/users/<input>` constrains the attacker to a subtree, but `../` climbs out of it to any endpoint on the same host:

```
../../admin/config
..%2f..%2fadmin%2fconfig        # encoded, to pass a literal ../ filter
..%252f..%252fadmin             # double-encoded, where one decode happens before the check
```

This does not change the host, so it is useful when the internal host is already the target and the goal is reaching a sensitive path (an admin route, an actuator or debug endpoint, an unauthenticated internal API) that the intended base path hid.

## Prefix allowlist bypass

A common control checks that the assembled URL starts with an allowed prefix, for example `https://storage.example.com/`. Traversal and normalization defeat a naive prefix match when the client normalizes the path after the check:

```
https://storage.example.com/../../internal/secret
https://storage.example.com/..%2f..%2finternal%2fsecret
```

The string passes `startsWith`, and the client collapses the `../` segments to reach a different path than the prefix implied.

## Authority and path parser differentials

URL parsers disagree on where the authority ends and the path begins, and that disagreement lets a value that one component reads as a path redirect the host another component connects to. The characters that drive this are `@`, `#`, `?`, and backslash:

```
http://allowed.example@127.0.0.1/          # userinfo trick: host is 127.0.0.1, not allowed.example
http://127.0.0.1#@allowed.example/         # fragment: real host is 127.0.0.1 before the #
http://127.0.0.1\@allowed.example/         # backslash: some parsers treat \ as /
http://allowed.example\@127.0.0.1/         # the reverse, depending on which parser validates
```

The attack is a differential: the validator parses the URL one way (seeing `allowed.example` as the host) while the HTTP client parses it another (connecting to `127.0.0.1`). WHATWG-style parsers treat backslash as a path separator and resolve userinfo differently from older RFC 3986 parsers, so a sink that validates with one library and fetches with another is the vulnerable combination. Which payload works depends on the exact pair, so cycle through the `@`, `#`, and `\` forms and watch which host the request reaches.

## Tools

- **Burp Repeater**: cycling encoded traversal and `@`/`#`/`\` authority payloads against the sink.
- **curl**: probing raw path and authority forms to see which host the request reaches.
- **SSRFmap**: automating prefix-allowlist and traversal payload delivery.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
