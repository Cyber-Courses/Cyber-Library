---
title: "Operator and CRD abuse: turning a controller's permissions against the cluster"
description: "Abusing a Kubernetes operator's broad permissions, or the custom resources it reconciles, to make the operator act for the attacker: creating privileged workloads, reading secrets, or granting RBAC, by crafting a custom resource the over-privileged controller then acts on."
keywords:
  - kubernetes operator
  - custom resource
  - CRD
  - controller permissions
  - confused deputy
---

# Operator and CRD abuse

Operators are controllers that watch custom resources and act on them, usually with wide permissions (create pods, read secrets, manage RBAC) so they can manage their software. An attacker who can create or edit the custom resources an operator reconciles makes the operator a confused deputy, performing privileged actions on the attacker's behalf.

```bash
kubectl get crds
kubectl get clusterrole -o json | jq -r '.items[]|select(.rules[]?.resources[]?=="secrets")|.metadata.name'  # operator roles
# Craft a custom resource whose reconciliation creates a privileged pod or reads a secret
```

## Exploitation notes

- The attacker needs only rights over the custom resource, not the operator's permissions; the operator supplies those when it acts.
- Target operators whose RBAC includes secrets, pods with privileged specs, or rolebinding creation.
- A custom resource that the operator renders into a pod spec is a path to [Pod creation to node](../rbac-privilege-escalation/pod-creation-to-node.md) without holding pod-create yourself.

## References

- [Kubernetes: operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- [Kubernetes: custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
