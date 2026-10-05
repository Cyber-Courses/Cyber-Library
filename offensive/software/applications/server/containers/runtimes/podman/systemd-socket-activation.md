---
title: "systemd socket activation: reaching the Podman socket through systemd"
description: "Reaching the Podman API socket exposed through systemd socket activation, in user or system scope, where the podman.socket unit starts the service on demand, so an attacker who can reach the socket path drives the engine without the service running beforehand."
keywords:
  - podman.socket
  - systemd socket activation
  - podman API
  - user socket
  - container runtime
---

# systemd socket activation

Podman ships `podman.socket` units for systemd, in both system and per-user scope. With socket activation the service is not running until something connects, but the socket file exists, so reaching it starts the service and drives the engine. The user-scope socket lives under the user's runtime directory.

```bash
systemctl --user status podman.socket 2>/dev/null
systemctl status podman.socket 2>/dev/null          # system scope (rootful)
ls -l /run/user/*/podman/podman.sock /run/podman/podman.sock 2>/dev/null

# Connecting activates the service
curl -s --unix-socket /run/user/1000/podman/podman.sock http://d/v1.40/libpod/info
```

## Exploitation notes

- System-scope `podman.socket` is rootful and root-equivalent; user-scope is bounded by that user.
- The socket path is the target: filesystem access to it (a shared runtime dir, a bind mount, a group membership) is engine access.
- Once connected, proceed as in [API service socket](api-service-socket.md).

## References

- [podman system service](https://docs.podman.io/en/latest/markdown/podman-system-service.1.html)
- [systemd.socket](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)
