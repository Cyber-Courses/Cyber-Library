---
title: "Kubelet credential theft: taking the node identity to reach the cluster"
description: "Widening a Kubernetes node foothold to the cluster by stealing the node's kubelet client certificate and kubeconfig, then acting as the node identity to read the secrets of every pod scheduled on it and, through node permissions, beyond."
keywords:
  - kubelet credentials
  - node identity
  - kubelet.conf
  - node authorization
  - cluster pivot
---

# Kubelet credential theft

Once on a node, the kubelet's credentials are the prize. The node authenticates to the API as `system:node:<name>` using a client certificate and kubeconfig on disk. Taking them lets an attacker act as the node, which the Node authorization mode allows to read the secrets of pods scheduled there, among other node-scoped rights.

```bash
cat /etc/kubernetes/kubelet.conf                      # node kubeconfig
ls /var/lib/kubelet/pki/                              # kubelet client cert/key
# Act as the node identity
kubectl --kubeconfig /etc/kubernetes/kubelet.conf auth can-i --list
```

## Exploitation notes

- The node identity can read secrets and configmaps of pods on that node; scheduling a target's workload onto a controlled node (or waiting for it) widens the reach.
- Node credentials plus the materialized secrets under `/var/lib/kubelet/pods/*/volumes/` sweep up every token mounted on the node.
- This is how one escaped pod becomes cluster-wide movement, feeding [Lateral movement](../lateral-movement/index.md).

## References

- [Kubernetes: using node authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
- [Kubernetes: kubelet authentication and authorization](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
