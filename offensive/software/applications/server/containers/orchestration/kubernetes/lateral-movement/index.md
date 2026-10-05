---
title: "Lateral movement: spreading across a Kubernetes cluster"
description: "Moving laterally in a Kubernetes cluster after a foothold: pivoting across the flat pod network to other workloads, stealing tokens and secrets to assume new identities, reusing service-account tokens against the API, and pivoting into the cloud through a pod or node identity."
keywords:
  - kubernetes lateral movement
  - pod pivoting
  - token theft
  - service account
  - cloud pivot
---

# Lateral movement

A single pod rarely holds everything. Lateral movement spreads the foothold: across the flat network to other pods, through stolen tokens and secrets into stronger identities, and out of the cluster into the cloud account the nodes and pods are tied to.

## Subtopics

- **[Pod to pod pivoting](pod-to-pod-pivoting.md)**: reaching other workloads over the pod network.
- **[Token and secret theft](token-and-secret-theft.md)**: collecting identities from pods and the node.
- **[Service account token to API](service-account-token-to-api.md)**: using a stolen token against the API.
- **[Cloud IAM via workload identity](cloud-iam-via-workload-identity.md)**: pivoting into the cloud account.

## References

- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [MITRE ATT&CK: Containers matrix](https://attack.mitre.org/matrices/enterprise/containers/)
