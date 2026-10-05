---
title: "CRI-O socket: driving the CRI runtime through crictl"
description: "Using the CRI-O or CRI runtime socket, reachable with crictl from a node foothold, to create and run privileged pods and containers that mount the host, root-equivalent because the runtime runs as root on the Kubernetes node."
keywords:
  - CRI-O socket
  - crictl
  - CRI runtime
  - kubernetes node
  - container runtime
---

# CRI-O socket

CRI-O exposes a CRI socket, by default `/var/run/crio/crio.sock`, driven by `crictl`. Like containerd it runs as root on the node, so reaching the socket creates privileged containers directly, bypassing the Kubernetes API and its admission controls entirely.

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///var/run/crio/crio.sock
crictl ps -a
crictl images

# Run a container from a pod/container spec that mounts the host and is privileged
crictl runp pod.json && crictl create <pod> ctr.json pod.json && crictl start <ctr>
```

## Exploitation notes

- Because this bypasses the API server, admission policies like Pod Security Standards do not apply; the socket is the trust boundary.
- `crictl` also works against containerd's CRI endpoint, so the same tool covers both runtimes on a node.
- The crafted container spec requests the host mount and privileged flag; the escape then follows [Privileged configuration](../../container-escape/privileged-configuration/index.md).

## References

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
