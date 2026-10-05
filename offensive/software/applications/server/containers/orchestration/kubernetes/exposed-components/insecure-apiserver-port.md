---
title: "Insecure apiserver port: the legacy unauthenticated API on 8080"
description: "Reaching a Kubernetes API server's legacy insecure port, which serves the full API with no authentication or authorization, giving complete cluster control to anyone who can connect when an old or misconfigured cluster still binds it."
keywords:
  - insecure port
  - 8080
  - kube-apiserver
  - unauthenticated API
  - legacy kubernetes
---

# Insecure apiserver port

Older kube-apiserver builds offered an insecure port (commonly `8080`) that served the entire API with no authentication and no authorization. Anyone who could reach it was effectively cluster-admin. It is removed in current Kubernetes, but appears on legacy clusters and misconfigured custom deployments.

```bash
curl -s http://<apiserver>:8080/version
curl -s http://<apiserver>:8080/api/v1/secrets            # all secrets, no auth
kubectl --server=http://<apiserver>:8080 get nodes
```

## Exploitation notes

- It is total compromise: no identity, no RBAC, full read and write of every resource including secrets and pods.
- It usually binds to localhost, so reaching it needs a node foothold or a [Host network namespace](../../../container-escape/shared-host-namespaces/host-network-namespace.md) pod; occasionally it is exposed more widely.
- Unlike [Anonymous API access](anonymous-api-access.md), there is no principal and no binding involved; the port itself is the flaw.

## References

- [Kubernetes: API server ports and arguments](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
