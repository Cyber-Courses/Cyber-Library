---
title: "Over-permissive roles: escalation through wildcards and dangerous verbs"
description: "Escalating in Kubernetes when a role grants more than intended: wildcard resources or verbs, access to secrets, the ability to create or exec into pods, or control over roles and bindings, each of which hands an attacker a direct path to a stronger identity."
keywords:
  - over-permissive role
  - RBAC wildcard
  - dangerous verbs
  - secrets access
  - kubernetes escalation
---

# Over-permissive roles

The most common escalation is simply a role that grants too much. Wildcards (`*` resources or verbs), `secrets` read, `pods/exec`, and write access to `roles` or `rolebindings` are each enough to climb. Enumerate your effective rights and look for these.

```bash
kubectl auth can-i --list
kubectl auth can-i get secrets
kubectl auth can-i create pods
kubectl auth can-i '*' '*'                 # wildcard = effectively admin in scope
```

## Exploitation notes

- `get secrets` yields other identities' tokens, often a shortcut straight to a more privileged service account.
- `create pods` or `pods/exec` escalates through the node, see [Pod creation to node](pod-creation-to-node.md).
- Write access to roles or bindings is self-escalation, bounded by the escalate and bind guards in [Escalate and bind verbs](escalate-and-bind-verbs.md).

## References

- [Kubernetes RBAC: role and clusterrole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)
- [Kubernetes: checking API access](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access)
