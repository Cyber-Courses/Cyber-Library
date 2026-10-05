---
title: "Cluster enumeration: mapping Kubernetes from a compromised pod"
description: "Enumerating a Kubernetes cluster from a foothold in a pod: discovering the mounted service-account token and what it can do, probing the API server and in-pod environment, reaching the cloud metadata service, and mapping services and the pod network."
keywords:
  - kubernetes enumeration
  - service account token
  - API server
  - in-pod recon
  - cluster mapping
---

# Cluster enumeration

A pod ships with everything needed to start enumerating: a service-account token, in-cluster DNS, and network reach to the API server and its neighbors. The first job is to learn who you are (what the token can do) and what is around you, before escalating.

## Subtopics

- **[In-pod enumeration](in-pod-enumeration.md)**: what the pod itself reveals.
- **[Service account and token discovery](service-account-and-token-discovery.md)**: find tokens and test them.
- **[API server enumeration](api-server-enumeration.md)**: probe the API with your identity.
- **[Service and network discovery](service-and-network-discovery.md)**: map services and the pod network.
- **[Cloud metadata from pod](cloud-metadata-from-pod.md)**: reach the node or workload cloud identity.

## References

- [Kubernetes API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [Kubernetes: access the API from a pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/)
