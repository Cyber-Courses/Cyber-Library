---
title: "SFTP scheme SSRF against internal endpoints"
description: "The sftp:// scheme makes a curl-backed client connect to an internal SSH file-transfer service, which probes port 22 and, where credentials are supplied or reused, reaches internal file transfer."
keywords:
  - sftp scheme
  - sftp://
  - port 22
  - internal file transfer
  - SSH
  - credential reuse
---

# SFTP

`sftp://` drives an SSH file-transfer connection from a curl-backed client. As an SSRF scheme its main value is reaching and probing an internal SSH service, with file transfer available only when the connection can authenticate.

## Connecting to an internal service

The URL carries host, optional port, and a path:

```
sftp://127.0.0.1:22/
sftp://10.0.0.5/etc/hostname
```

The client opens a connection to port 22 (or the given port) and begins the SSH handshake. Because the connection succeeds, fails, or hangs distinctly, `sftp://` is a clean oracle for whether an internal host runs SSH, complementing a [Port](../authority/port.md) sweep with a protocol-aware probe.

## Authentication is the limit

Unlike [File](file.md) or [Gopher](gopher.md), SFTP requires a successful SSH authentication before any file is read, so a bare `sftp://` to an internal host confirms the service but does not by itself return data. It becomes a file-read path when the sink lets credentials ride in the URL or the environment, or when the client is configured to reuse a key or an agent, in which case supplying a known or reused credential reaches internal transfer:

```
sftp://user:pass@10.0.0.5/home/user/.ssh/id_rsa
```

Where no credential is available, treat `sftp://` as a service-presence and reachability oracle rather than a disclosure primitive.

## Tools

- **curl**: manual `sftp://` probing to test whether an internal host speaks SSH on port 22.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
