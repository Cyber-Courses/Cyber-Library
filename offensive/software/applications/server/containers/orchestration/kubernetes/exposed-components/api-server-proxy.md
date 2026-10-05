---
title: "API server proxy: reaching internal services through the API proxy"
description: "Abusing the Kubernetes API server's proxy subresources, or kubectl proxy, to reach cluster-internal services, pods, and node components from a client with a usable identity, pivoting to endpoints that assume they are only reachable from inside the cluster."
keywords:
  - API server proxy
  - kubectl proxy
  - proxy subresource
  - internal services
  - kubernetes pivot
---

# API server proxy

The API server can forward requests to services, pods, and nodes through its proxy subresources, and `kubectl proxy` exposes the same capability locally. An identity allowed to use these subresources turns the API server into a gateway to internal services, dashboards, and kubelets that were never meant to be reachable from outside the cluster.

```bash
# Proxy to an internal service through the API server
kubectl proxy --port=8001 &
curl -s http://127.0.0.1:8001/api/v1/namespaces/<ns>/services/<svc>:<port>/proxy/

# Direct proxy-subresource URL form (pod, service, or node)
curl -sk -H "Authorization: Bearer $TOKEN" \
  $API/api/v1/namespaces/<ns>/pods/<pod>/proxy/
```

## Exploitation notes

- The escalation is turning a token that can `get` the proxy subresource into reach of internal-only endpoints, including the kubelet and node proxies.
- Chain it to hit internal dashboards, metrics, and admin UIs that trust the cluster network.
- This is the API server proxying, distinct from kube-proxy, which only programs node Service networking and is not an HTTP gateway; for reaching peers over the pod network see [Pod to pod pivoting](../lateral-movement/pod-to-pod-pivoting.md).

## References

- [Kubernetes: access services running on clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/)
- [Kubernetes: kubectl proxy](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#proxy)
