---
title: "Pod escape to node: breaking out of a pod onto its node"
description: "Escaping a Kubernetes pod to the node it runs on by scheduling or using a pod with a privileged security context, a hostPath mount, or shared host namespaces, then stealing the node's kubelet credentials to pivot from one node to the whole cluster."
keywords:
  - pod escape
  - node breakout
  - privileged pod
  - hostPath
  - kubelet credentials
---

# Pod escape to node

Escaping a pod to its node is the same container breakout as anywhere, delivered through a pod spec. Kubernetes decides what a pod may request (privileged, hostPath, host namespaces) through admission control, so the escape is really about obtaining or using a pod with a dangerous spec. Once on the node, the kubelet's credentials turn one node into cluster-wide reach. The breakout primitives themselves live under [Container escape](../../../container-escape/index.md).

## Subtopics

- **[Privileged pod](privileged-pod.md)**: a pod with a privileged security context.
- **[hostPath mount](hostpath-mount.md)**: a pod mounting a node path.
- **[Host namespaces](host-namespaces.md)**: a pod sharing hostPID, hostNetwork, or hostIPC.
- **[Kubelet credential theft](kubelet-credential-theft.md)**: taking the node identity to reach the cluster.

## References

- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Kubernetes: security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
