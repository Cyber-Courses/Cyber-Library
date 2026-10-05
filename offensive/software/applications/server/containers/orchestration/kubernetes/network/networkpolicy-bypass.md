---
title: "NetworkPolicy bypass: reaching pods a policy meant to isolate"
description: "Reaching Kubernetes pods and services a NetworkPolicy was intended to isolate, through a missing default-deny, gaps in namespace or label selectors, traffic the policy does not cover such as DNS and node-local endpoints, or a CNI that does not enforce policy at all."
keywords:
  - NetworkPolicy bypass
  - default deny
  - CNI enforcement
  - network isolation
  - kubernetes
---

# NetworkPolicy bypass

NetworkPolicy is allow-listing that only works if it is complete and enforced. The common gaps are a missing default-deny (so unmatched traffic is allowed), selectors that do not cover every path, protocols the policy omits, and, critically, a CNI that does not implement NetworkPolicy, in which case the objects exist but nothing enforces them.

```bash
kubectl get networkpolicies -A
# Does the CNI enforce policy at all? Test reachability the policy should block.
kubectl run t --image=alpine --rm -it -- sh -c 'nc -z -w2 <blocked-pod-ip> <port>; echo $?'
```

## Exploitation notes

- Without a default-deny ingress and egress, any path not explicitly denied is open; most clusters never add one.
- Some CNIs ignore NetworkPolicy entirely, so verify enforcement empirically rather than trusting the objects.
- Egress policies are frequently missing, leaving the cloud metadata endpoint and external callbacks reachable.

## References

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: default deny network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-deny-all-ingress-and-all-egress-traffic)
