---
title: "Enumeration: fingerprinting Grafana"
description: "Grafana exposes its version on the login page, the /api/health endpoint, and static resources, usually on port 3000. The version is decisive because the unauthenticated path-traversal and other vulnerabilities are version-specific, so reading the exact build directs the choice of exploit before any authentication."
keywords:
  - grafana enumeration
  - api/health
  - version
  - port 3000
  - fingerprint
---

# Enumeration

Grafana is easy to fingerprint and the version is decisive. It commonly listens on port 3000 (often reverse-proxied onto 80/443), and the login page, the unauthenticated `/api/health` endpoint, and the versioned static resources all disclose the exact build. That version matters because the headline unauthenticated path-traversal file-read and other plugin/server vulnerabilities apply only to specific ranges, so reading it first tells you whether the instance is exploitable without credentials and which issues to pursue.

```bash
# version without authentication
curl -sk https://<target>:3000/api/health            # {"database":"ok","version":"x.y.z",...}
curl -sk https://<target>:3000/login | grep -ioE 'Grafana v[0-9.]+'
```

## Exploitation notes

- `/api/health` returns the version unauthenticated; it maps directly to the [path-traversal](path-traversal-file-read.md) and [known exploits](known-exploits.md) applicability.
- The version decides whether the unauthenticated file-read is present, which changes whether you need credentials at all.
- Grafana is frequently exposed on 3000 or behind a proxy; scan for it and read the version regardless of the front.
- Route to [authentication](authentication-and-anonymous-access.md), the unauthenticated traversal, or the data-source routes depending on access and version.

## References

- [Grafana: HTTP API health](https://grafana.com/docs/grafana/latest/developers/http_api/other/)
- [Grafana security advisories](https://grafana.com/security/security-advisories/)
