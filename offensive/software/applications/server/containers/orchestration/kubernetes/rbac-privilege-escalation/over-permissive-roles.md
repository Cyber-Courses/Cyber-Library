---
title: "Over-permissive roles: escalation through broad grants"
description: "Roles that grant wildcard verbs or resources, or the ability to read secrets across namespaces, hand an identity far more than intended. Reading all secrets yields other identities' tokens and application credentials, and a wildcard role on a namespace or the cluster is effectively admin over that scope, needing no further trick."
keywords:
  - rbac
  - wildcard role
  - get secrets
  - clusterrole
  - privilege escalation
---

# Over-permissive roles

The simplest RBAC escalation is a role that is just too broad. Operators grant `*` verbs or `*` resources for convenience, or allow `get`/`list` on `secrets` cluster-wide, and that single grant is the escalation. Reading secrets exposes every other identity's service-account token and every application credential stored as a secret; a wildcard role is admin over its scope outright.

Find the broad grants the identity has:

```bash
kubectl auth can-i --list                           # look for *, secrets, and cluster scope
kubectl auth can-i get secrets --all-namespaces
kubectl auth can-i '*' '*' --all-namespaces
```

## Reading secrets to harvest identities

```bash
# dump every secret and decode it; token-type secrets are other SA tokens
kubectl get secrets --all-namespaces -o json | python3 -c '
import sys,json,base64
for s in json.load(sys.stdin)["items"]:
    for k,v in (s.get("data") or {}).items():
        try: val=base64.b64decode(v).decode("utf-8","replace")
        except Exception: continue
        if k=="token" or any(x in k.lower() for x in ("pass","key","secret")):
            print(s["metadata"]["namespace"], s["metadata"]["name"], k, val[:60])'
# a recovered SA token for a powerful account is used directly against the API
```

A `kubernetes.io/service-account-token` secret contains a non-expiring token for its service account; reading one for a high-privilege account (a controller, or anything bound to `cluster-admin`) is immediate escalation.

## Exploitation notes

- `get secrets` cluster-wide is effectively cluster compromise, because it exposes the tokens of every service account, including admin-bound ones.
- Wildcard roles (`verbs: ["*"]`, `resources: ["*"]`) grant the escalation verbs (`escalate`, `bind`, `impersonate`) implicitly; treat a wildcard as access to all the other routes in this group.
- Namespaced breadth still matters: a wildcard Role in `kube-system` reaches controller service accounts whose tokens are cluster-powerful.
- Feed recovered tokens into [Service account token to API](../lateral-movement/service-account-token-to-api.md).

## References

- [Kubernetes: RBAC good practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
- [Kubernetes: secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks: Kubernetes RBAC](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
