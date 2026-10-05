---
title: "Escalate and bind verbs: granting yourself more through RBAC"
description: "Escalating in Kubernetes with the escalate and bind RBAC verbs, which let an identity create or attach to roles that carry permissions it does not already hold, bypassing the normal rule that you cannot grant more than you have."
keywords:
  - escalate verb
  - bind verb
  - RBAC escalation
  - clusterrolebinding
  - kubernetes privilege
---

# Escalate and bind verbs

Normally RBAC stops you granting permissions you do not hold. Two verbs override that. `escalate` lets you write a role with permissions beyond your own; `bind` lets you create a binding to a role more powerful than you hold. Either one, on roles or bindings, is a path to cluster-admin.

```bash
kubectl auth can-i escalate clusterroles
kubectl auth can-i bind clusterroles

# With bind: attach yourself (or a controlled SA) to cluster-admin
kubectl create clusterrolebinding pwn --clusterrole=cluster-admin \
  --serviceaccount=<ns>:<sa>
```

## Exploitation notes

- `bind` plus an existing powerful ClusterRole (like `cluster-admin`) is the fastest route: no new role needed, just a binding.
- `escalate` is used when no suitable role exists: author one with the permissions you want, then bind it.
- These verbs are rarely granted deliberately; they usually arrive through a wildcard in [Over-permissive roles](over-permissive-roles.md).

## References

- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
