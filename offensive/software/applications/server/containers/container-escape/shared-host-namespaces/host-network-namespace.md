---
title: "Host network namespace: reaching host interfaces and services"
description: "Abusing a container that shares the host network namespace to reach services bound to the host's loopback interface, sniff and spoof on the host's real interfaces, and attack node-local components that trust localhost."
keywords:
  - hostNetwork
  - loopback services
  - container network
  - container escape
  - localhost trust
---

# Host network namespace

Sharing the host network namespace (`--net=host`, Kubernetes `hostNetwork`) puts the container on the host's interfaces, including `lo`. That reaches services bound to `127.0.0.1` that assume only the host can talk to them, which on a node often includes the kubelet, metrics, and admin endpoints.

```bash
ip addr                                   # host interfaces, not a veth pair
ss -ltnp | grep 127.0.0.1                 # loopback-bound host services
curl -sk https://127.0.0.1:10250/pods     # e.g. the kubelet read API on a node
```

## Exploitation notes

- The highest-value targets are localhost-trusting services: the kubelet (`10250`), cloud agents, and local admin sockets that skip authentication for loopback.
- Combined with [CAP_NET_RAW](../privileged-configuration/capability-abuse/cap-net-raw.md), it allows sniffing and spoofing on the host's real segments.
- It also exposes the cloud metadata service as the node sees it, covered under [Cloud metadata from pod](../../orchestration/kubernetes/cluster-enumeration/cloud-metadata-from-pod.md).

## References

- [man 7 network_namespaces](https://man7.org/linux/man-pages/man7/network_namespaces.7.html)
- [Kubernetes: kubelet authentication](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
