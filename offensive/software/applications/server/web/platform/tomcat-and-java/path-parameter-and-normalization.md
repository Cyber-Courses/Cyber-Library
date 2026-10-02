---
title: "Servlet path parameters and normalization: ..;/ traversal and constraint bypass"
description: "Exploiting Java servlet path handling: the ;path-parameter separator and normalization differences that bypass security-constraint URL patterns and reach protected servlets and management apps."
keywords:
  - path parameters
  - ..;/
  - servlet security constraint
  - tomcat normalization
  - access control bypass
---

# Path parameters and normalization

Java servlet containers treat `;` as the start of a **path parameter** (a matrix parameter) within a path segment, and they normalize paths with their own rules. Security constraints in `web.xml` (and front-proxy rules) are matched against URL patterns, so a path that normalizes differently on each side bypasses the constraint while still resolving to the protected servlet.

## The ;path-parameter separator

A segment like `admin;x=y` has the `;x=y` stripped as a path parameter after routing decisions, so `..;/` is a traversal segment that many front rules and some container checks do not treat as `..`:

```
GET /app/..;/manager/html HTTP/1.1          # reach /manager past a /app-scoped rule
GET /;/admin/ HTTP/1.1
GET /app/js/..;/..;/admin HTTP/1.1
```

This is a classic way to reach the [Tomcat Manager](manager-and-host-manager.md) or other protected contexts when a reverse proxy only allowed `/app` through: the proxy sees a path under `/app`, the container strips the path parameters and normalizes to `/manager`.

## Security-constraint pattern bypass

`web.xml` `<security-constraint>` matches `<url-pattern>` values. Mismatches between the pattern matcher and the dispatcher let crafted paths dodge a constraint:

- Trailing or extra characters (`/admin/`, `/admin%20`, `/admin.`) that the constraint pattern does not match but the servlet mapping does.
- Case differences where the constraint is case-sensitive but the mapping is not (depends on container/OS).
- Encoded separators (`%2e`, `%2f`, `%3b` for `;`) that the constraint check does not decode but the container does.

## Exploitation

- To reach a context blocked at the edge or by a constraint, insert `..;/` to climb and redirect to the target context (`/manager/html`, `/host-manager/html`, internal servlets).
- Combine with the [reverse-proxy normalization mismatch](../reverse-proxy-and-edge/normalization-mismatch.md): the proxy and Tomcat disagree on where `..;/` points.
- Probe constraint patterns with trailing/encoded variants and compare authenticated vs unauthenticated responses.

## Tools

- **curl --path-as-is** / Burp Intruder with `;`/encoding payloads.

## References

- Apache Tomcat: security constraints, path parameter handling
- Orange Tsai: Java/servlet path confusion research
