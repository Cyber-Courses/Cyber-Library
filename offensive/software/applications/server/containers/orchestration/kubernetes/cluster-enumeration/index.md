---
title: "Cluster enumeration: mapping a Kubernetes cluster from a foothold"
order: 1
description: "The first moves after landing in a Kubernetes pod: inventory what the pod already holds (its service-account token, mounted secrets, environment), discover the service-account identity and what it can do against the API server, enumerate services and the pod network, and reach the cloud metadata endpoint for node credentials."
keywords:
  - kubernetes enumeration
  - service account token
  - api server
  - cluster recon
  - pod
---

# Cluster enumeration

A foothold in Kubernetes is almost always a shell in a pod. The pod is not a bare container: it was given an identity and a view of the cluster, and enumerating both is what turns the foothold into movement. The order that pays off is to inventory the pod itself first (the service-account token and any mounted secrets are frequently enough to act immediately), then test what that identity can do against the API server, then map the services and network, and finally reach the cloud metadata endpoint for the node's own credentials.

The two endpoints every pod can see:

```bash
# the in-cluster API server, from injected environment
echo "$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"      # usually 10.96.0.1:443
# the pod's own service-account token, CA, and namespace
ls /var/run/secrets/kubernetes.io/serviceaccount/            # token  ca.crt  namespace
```

## Subtopics

- **[In-pod enumeration](in-pod-enumeration.md)**: secrets, tokens, and environment inside the pod.
- **[Service account and token discovery](service-account-and-token-discovery.md)**: the pod's identity and other reachable tokens.
- **[API server enumeration](api-server-enumeration.md)**: what the identity can read and do against the API.
- **[Service and network discovery](service-and-network-discovery.md)**: services, endpoints, DNS, and the pod network.
- **[Cloud metadata from pod](cloud-metadata-from-pod.md)**: reaching the node's cloud instance credentials.

## References

- [Kubernetes API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
- [HackTricks: Kubernetes enumeration](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
- [NCC Group: Kubernetes security](https://research.nccgroup.com/)
