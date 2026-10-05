---
title: "Lateral movement: spreading across a cluster and into the cloud"
description: "After a foothold, movement across a Kubernetes cluster follows its trust relationships: a service-account token used against the API server, secrets and tokens stolen from other namespaces, pivoting pod to pod across the flat network, and following a pod's workload identity out into the cloud account. Each step reuses credentials the cluster hands out."
keywords:
  - lateral movement
  - service account token
  - pod pivoting
  - workload identity
  - kubernetes
---

# Lateral movement

Lateral movement in Kubernetes is credential reuse along the cluster's own trust edges. A pod holds a service-account token that authenticates to the API server; the API exposes secrets and more tokens; the flat pod network reaches other workloads directly; and a pod's cloud workload identity reaches the surrounding cloud account. Each move takes a credential the cluster issued for a legitimate purpose and uses it to reach the next target, so movement rarely needs an exploit, only enumeration and reuse.

```bash
# the token you hold, what it reaches, and the network around you
kubectl auth can-i --list
kubectl get secrets --all-namespaces 2>/dev/null | head
kubectl get endpoints -A 2>/dev/null | head
```

## Subtopics

- **[Service account token to API](service-account-token-to-api.md)**: authenticating and acting with a pod token.
- **[Token and secret theft](token-and-secret-theft.md)**: harvesting credentials across namespaces.
- **[Pod-to-pod pivoting](pod-to-pod-pivoting.md)**: reaching other workloads over the flat network.
- **[Cloud IAM via workload identity](cloud-iam-via-workload-identity.md)**: following a pod identity into the cloud account.

## References

- [Kubernetes: controlling access](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
- [HackTricks: Kubernetes lateral movement](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
