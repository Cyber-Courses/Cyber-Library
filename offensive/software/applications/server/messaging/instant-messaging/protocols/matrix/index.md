---
title: "Matrix: attacking homeservers over the client-server and federation APIs"
description: "Matrix homeservers (Synapse, Dendrite, Conduit) expose an HTTP/JSON client-server API on 8008 or 443 and a federation API on 8448, with .well-known delegation pointing clients and servers at the real host. The surface is unauthenticated version and registration probing, open registration and shared-secret admin registration, media-repository SSRF, and federation request abuse."
keywords:
  - Matrix
  - homeserver
  - Synapse
  - federation
  - client-server API
---

# Matrix

Matrix is a federated messaging protocol spoken as HTTP with JSON bodies. A homeserver (Synapse is the reference implementation; Dendrite and Conduit are alternatives) serves the client-server API on TCP 8008 behind a reverse proxy on 443, and the server-to-server federation API on 8448. Clients and federating servers find the real host through `.well-known` delegation files. Because everything is HTTP, the whole surface is reachable with `curl`: you can fingerprint the server, probe whether registration is open, test for the Synapse admin API, search the user and room directories, and query profiles over federation, all without credentials.

## Triage

```bash
# fingerprint and delegation, no auth needed
curl -s https://target.lan/_matrix/client/versions          # supported spec versions + "unstable_features"
curl -s https://target.lan/.well-known/matrix/server         # federation delegation target:port
curl -s https://target.lan/.well-known/matrix/client         # client API base_url
curl -s https://target.lan:8448/_matrix/federation/v1/version
# the federation version body names the software, e.g. {"server":{"name":"Synapse","version":"1.xx.x"}}
```

The `/_matrix/federation/v1/version` body names Synapse, Dendrite, or Conduit and its version, which decides the server-side flaws that apply. The `.well-known` files reveal the true backend host and port when the service is fronted by a proxy.

## Triage questions

- Is registration open (`/_matrix/client/v3/register`), and is guest access enabled? That drives [authentication](authentication.md).
- Is the Synapse admin API reachable, and does the deployment use shared-secret registration you can abuse to mint an admin? Also [authentication](authentication.md).
- Can you search the user and room directories and query profiles over federation without an account? That drives [enumeration](enumeration.md).
- Which homeserver and version is it, and is it a Synapse build with media-repository URL-preview SSRF? That drives [server exploitation](server-exploitation.md).

## Subtopics

- **[Enumeration](enumeration.md)**: `.well-known` delegation, registration and admin-API probing, user-directory and public-rooms search, and federation profile queries.
- **[Authentication](authentication.md)**: open registration, guest access, SSO/token flows, and abusing Synapse shared-secret registration to create an admin account.
- **[Server exploitation](server-exploitation.md)**: Synapse media-repository URL-preview SSRF to internal services and cloud metadata, federation request forgery and resource exhaustion, and Dendrite/Conduit parsing bugs.

## References

- [Matrix specification](https://spec.matrix.org/latest/)
- [Matrix client-server API](https://spec.matrix.org/latest/client-server-api/)
- [Synapse homeserver](https://github.com/element-hq/synapse)
