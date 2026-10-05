---
title: "kube-proxy: reaching internal services through an exposed proxy"
description: "Abusing an exposed kube-proxy or the API server's proxy endpoints to reach cluster-internal services, dashboards, and node components from outside the cluster network, pivoting to services that assume they are only reachable internally."
keywords:
  - kube-proxy
  - API proxy
  - internal services
  - service exposure
  - kubernetes pivot
---

# kube-proxy

kube-proxy programs node networking, and the API server offers proxy endpoints that forward to services, pods, and nodes. Where the API proxy is reachable with a usable identity, or a proxy is otherwise exposed, it becomes a gateway to internal services, dashboards, and kubelets that were never meant to be reachable from outside.

```bash
# Proxy to an internal service through the API server
kubectl proxy --port=8001 &
curl -s http://127.0.0.1:8001/api/v1/namespaces/<ns>/services/<svc>:<port>/proxy/

# Direct API proxy URL form
curl -sk -H "Authorization: Bearer $TOKEN" \
  $API/api/v1/namespaces/<ns>/pods/<pod>/proxy/
```

## Exploitation notes

- The proxy turns any service-account token that can `get` the proxy subresource into reach of internal-only endpoints.
- Chain it to hit internal dashboards, metrics, and admin UIs that trust the cluster network.
- For reaching peers without the proxy, see [Pod to pod pivoting](../lateral-movement/pod-to-pod-pivoting.md).

## References

- [Kubernetes: kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
- [Kubernetes: access services running on clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/)
