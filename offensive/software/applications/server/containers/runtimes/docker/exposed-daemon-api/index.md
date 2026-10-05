---
title: "Exposed daemon API: reaching the root-equivalent Docker daemon"
description: "Reaching the Docker daemon over an exposed API, on the plaintext port, a TLS port with weak or unverified client certificates, or a mounted socket, which grants full root-equivalent control of the host because the daemon can start a privileged, host-mounting container."
keywords:
  - docker daemon API
  - 2375
  - 2376
  - docker remote API
  - container takeover
---

# Exposed daemon API

The Docker daemon API is root on the host by design: anyone who can talk to it can start a privileged container that mounts the host. It listens on a local socket by default, but operators frequently expose it on TCP, on `2375` in plaintext or `2376` with TLS, for remote management and CI. Any reachable daemon API is game over for the host.

## Subtopics

- **[Unauthenticated daemon access](unauthenticated-daemon-access.md)**: the plaintext API on 2375.
- **[Weak TLS on 2376](weak-tls-on-2376.md)**: the TLS port without enforced client certificates.
- **[API enumeration](api-enumeration.md)**: mapping the host through the API.
- **[Host takeover via privileged run](host-takeover-via-privileged-run.md)**: turning API access into host root.

## References

- [Docker Engine API](https://docs.docker.com/engine/api/)
- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
