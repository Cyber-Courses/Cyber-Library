---
title: "In-pod enumeration: what a compromised pod reveals"
description: "Enumerating a Kubernetes cluster from inside a pod using only local signals: the mounted service-account token and namespace, environment variables with service addresses, mounted secrets and config maps, and the node and cluster hints in the pod's filesystem and metadata."
keywords:
  - in-pod enumeration
  - service account mount
  - kubernetes environment
  - mounted secrets
  - pod recon
---

# In-pod enumeration

Before any network call, the pod itself leaks a lot. The service-account mount identifies the namespace and provides a token, environment variables expose service addresses, and mounted secrets and config maps often hold the credentials you came for.

```bash
# Identity and namespace
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
ls /var/run/secrets/kubernetes.io/serviceaccount/

# Service addresses injected by Kubernetes, and anything mounted in
env | grep -iE 'SERVICE_|KUBERNETES_'
mount | grep -i secret; find / -maxdepth 6 -name '*.yaml' -path '*config*' 2>/dev/null
```

## Exploitation notes

- The `serviceaccount` mount gives the token, CA certificate, and namespace with no network call; it is the starting identity.
- Injected `*_SERVICE_HOST` variables map the services the pod was meant to reach, a ready target list.
- Mounted secrets and config maps are the quickest credential win; sweep every mount before probing the API.

## References

- [Kubernetes: access the API from a pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/)
- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
