---
title: "SSRF URL schemes: gopher, file, dict, ldap, and handlers beyond HTTP in server-side fetches"
description: Non-http schemes in server-side fetches, gopher, dict, file, ldap, and legacy URL handlers that change the protocol handler.
keywords:
  - SSRF
  - URL scheme
  - gopher
  - file URI
---

# URL schemes (SSRF)

The scheme (`http:`, `https:`, `gopher:`, `file:`, …) selects which handler runs in the URL client. Many SSRF bugs come from policy that only mentions HTTP while the underlying library still accepts `gopher://` or `file://` when passed an unvalidated string.

## Pages

| Page | Notes |
|------|--------|
| [Gopher](gopher.md) | Line-based payloads; smuggling-like abuse on some stacks |
| [File](file.md) | Local file read via `file://` handlers |
| [DICT](dict.md) | `dict://host:port/` minimal protocol abuse |
| [LDAP](ldap.md) | `ldap://` and directory-oriented clients |
| [TFTP](tftp.md) | UDP-based; rare in app HTTP clients |
| [SFTP](sftp.md) | Often via SSH libraries, not raw URL |
| [JAR](jar.md) | Java-specific handler chains |
| [Netdoc](netdoc.md) | Legacy / platform-specific |
| [HTTP SSRF port scanning](http-ssrf-port-scanning.md) | Using `http` to probe ports on one host |
