---
title: "Token and secret theft: collecting identities across the cluster"
description: "Collecting credentials across a Kubernetes cluster after a foothold: service-account tokens mounted into pods, secrets readable through the API or on the node, and the materialized secret volumes on a node, each granting a new identity to move with."
keywords:
  - token theft
  - secret theft
  - mounted secrets
  - kubelet secrets
  - lateral movement
---

# Token and secret theft

Credentials are scattered across a cluster: every pod mounts a service-account token, secrets hold application and registry credentials, and a node materializes the secrets of every pod it runs. Sweeping them up yields the identities to move laterally and escalate.

```bash
# From API access: read secrets and token-type secrets
kubectl get secrets -A -o json | jq -r '.items[]|.metadata.namespace+"/"+.metadata.name+" "+.type'

# From a node foothold: every mounted secret on the node
find /var/lib/kubelet/pods -path '*volumes/kubernetes.io~secret/*' -type f 2>/dev/null
```

## Exploitation notes

- A node holds the secrets of all pods scheduled on it, so one node foothold often harvests many identities at once.
- Prioritize tokens whose service accounts hold broad RBAC; test each with `auth can-i --list`.
- Feed the strongest token into [Service account token to API](service-account-token-to-api.md) and the registry creds into image access.

## References

- [Kubernetes: secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
