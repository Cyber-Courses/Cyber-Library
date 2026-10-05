---
title: "TokenRequest API: minting tokens for service accounts you can act on"
description: "Escalating in Kubernetes with the TokenRequest API or the create serviceaccounts/token right, which mints a valid token for a service account, so an identity able to request tokens for a privileged account can act as it."
keywords:
  - TokenRequest API
  - serviceaccounts/token
  - token minting
  - service account
  - kubernetes escalation
---

# TokenRequest API

The TokenRequest API issues short-lived tokens for service accounts. The permission `create` on the `serviceaccounts/token` subresource lets an identity mint a token for that account. If you can request tokens for a more privileged service account, you can become it.

```bash
kubectl auth can-i create serviceaccounts/token
# Mint a token for a target service account
kubectl create token <privileged-sa> -n <ns>
kubectl --token=<minted> auth can-i --list
```

## Exploitation notes

- This is the modern replacement for reading long-lived token secrets; the right to mint is the escalation, scoped to the service accounts you may act on.
- Combine with enumeration of which service accounts are privileged, then mint for the strongest one you are allowed.
- Minted tokens are time-bound, so use them promptly or re-mint; for durable access prefer [CSR approval](csr-approval.md).

## References

- [Kubernetes: service account token volume projection](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#serviceaccount-token-volume-projection)
- [Kubernetes: TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/)
