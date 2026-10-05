---
title: "NetworkPolicy bypass: reaching targets a policy was meant to isolate"
description: "NetworkPolicies are allowlists enforced by the CNI, and they fail in predictable ways: namespaces with no policy are fully open, policies often omit egress or only cover some pods, DNS and the API server are commonly left reachable, and node-level or host-network paths sidestep pod-scoped rules. An attacker maps the gaps and routes around the intended isolation."
keywords:
  - networkpolicy
  - egress
  - cni enforcement
  - dns
  - bypass
---

# NetworkPolicy bypass

A NetworkPolicy is an allowlist of permitted traffic for the pods it selects, enforced by the CNI. Its weaknesses are structural rather than exploit-based. A policy applies only to the pods its selector matches, so pods and whole namespaces with no policy are unrestricted. Policies frequently define ingress but omit egress, or vice versa. They commonly leave DNS and the API server reachable because workloads need them. And rules scoped to pod traffic do not constrain node-level or host-network paths. Mapping these gaps is how an attacker reaches a target the policy was supposed to isolate.

Find the gaps:

```bash
# which namespaces and pods have no policy at all?
kubectl get networkpolicies -A
kubectl get ns -o name | while read ns; do \
  n=$(kubectl get netpol -n ${ns#namespace/} --no-headers 2>/dev/null | wc -l); \
  echo "${ns#namespace/}: $n policies"; done
# does a given policy cover egress, or only ingress?
kubectl get netpol -n <ns> -o yaml | grep -E 'policyTypes|Ingress|Egress'
```

## Routes around a policy

```bash
# 1. a namespace or pod with no policy is fully reachable; pivot through it
# 2. egress not restricted: exfiltrate and reach external or other-namespace targets
# 3. DNS left open: tunnel over DNS, or use allowed resolver paths
# 4. host-network path: a hostNetwork pod is on the node stack, outside pod-scoped rules
#    (see Host namespaces) so it reaches node-local and policy-exempt destinations
# 5. the CNI may not enforce policy on certain traffic (e.g. node->pod, or hairpin)
```

## Exploitation notes

- The first check is coverage: any namespace with zero policies is open, and clusters commonly protect a few sensitive namespaces while leaving the rest flat, so a neighbour namespace is often an unrestricted pivot.
- Egress omissions are the most common functional gap; a policy that only filters ingress still lets a compromised pod reach everything outward, including other-namespace services by IP.
- A `hostNetwork` pod escapes pod-scoped policy entirely because its traffic originates from the node; combine with [Host namespaces](../pod-escape-to-node/host-namespaces.md).
- Policy enforcement depends on the CNI actually implementing NetworkPolicy; some configurations accept the objects but do not enforce them, which `kubectl get netpol` cannot reveal, so test reachability directly.

## References

- [Kubernetes: network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes: network policy gotchas](https://kubernetes.io/docs/concepts/services-networking/network-policies/#what-you-can-t-do-with-network-policies-currently)
