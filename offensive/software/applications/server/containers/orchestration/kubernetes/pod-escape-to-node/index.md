---
title: "Pod escape to node: breaking out of a pod onto its worker"
order: 4
description: "A pod that is over-privileged escapes to its worker node with no exploit: a privileged pod, a hostPath mount of the node filesystem, shared host namespaces, or access to the kubelet's credentials each give node root. Owning the node then exposes every other pod's secrets and the kubelet credentials that pivot to the cluster."
keywords:
  - pod escape
  - worker node
  - privileged pod
  - hostpath
  - kubelet
---

# Pod escape to node

A pod is a set of containers on a worker node, and the same misconfigurations that make a container escapable make a pod escape to its node. The routes are the Kubernetes expression of the container-escape primitives: a pod with `privileged: true`, a `hostPath` volume mounting the node filesystem, shared host namespaces, or reachable kubelet credentials. Escaping to the node is high-value because the node runs every pod scheduled to it, so node root exposes all their secrets and tokens, and the node's own kubelet identity pivots toward the cluster.

Check the pod's security context for the easy routes:

```bash
grep -E 'CapEff|Seccomp' /proc/self/status; cat /proc/self/uid_map
mount | grep -vE 'overlay|proc|sysfs|tmpfs|cgroup' | head    # hostPath mounts
ls /dev | grep -E 'sd|nvme' ; ls /var/run/secrets/kubernetes.io/serviceaccount/
readlink /proc/1/exe                                         # host init => shared PID ns
```

## Subtopics

- **[Privileged pod](privileged-pod.md)**: a pod with the privileged security context.
- **[hostPath mount](hostpath-mount.md)**: a volume mounting the node filesystem.
- **[Host namespaces](host-namespaces.md)**: hostPID, hostNetwork, and hostIPC pods.
- **[Kubelet credential theft](kubelet-credential-theft.md)**: stealing the node's kubelet identity.

## References

- [Kubernetes: pod security standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [BishopFox: bad pods](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Container escape (runtime-agnostic)](../../../container-escape/index.md)
