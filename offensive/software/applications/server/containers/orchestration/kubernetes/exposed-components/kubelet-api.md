---
title: "Kubelet API: executing in pods through the node agent"
description: "Reaching the kubelet API on a node, which when it allows anonymous or unauthorized access lists the pods on the node and runs commands inside them, giving code execution in any container on the node without going through the API server."
keywords:
  - kubelet API
  - 10250
  - kubelet exec
  - node agent
  - pod command execution
---

# Kubelet API

Every node runs a kubelet that exposes an API, usually on `10250`, to manage the node's pods. Where it allows anonymous access or the node's authorization is lax, it lists pods and runs commands inside them directly, which is code execution in any container on the node, bypassing the API server and its RBAC.

```bash
K=https://<node>:10250
curl -sk $K/pods | jq '.items[].metadata | {namespace,name}'
# Run a command in a container (namespace, pod, container from /pods)
curl -sk -X POST "$K/run/<ns>/<pod>/<container>" -d "cmd=id"
```

## Exploitation notes

- `/pods` is the inventory; `/run` and `/exec` give execution inside those containers, often including more privileged workloads than your own.
- Target pods with powerful service-account tokens or host mounts; exec in, then steal the token or use the mount.
- Reaching `10250` needs node network access, from a node foothold or a [Host network namespace](../../../container-escape/shared-host-namespaces/host-network-namespace.md) pod.

## References

- [Kubernetes: kubelet authentication and authorization](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [Kubernetes: ports and protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
