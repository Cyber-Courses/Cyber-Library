---
title: "API server proxy: reaching nodes and services through the control plane"
description: "The API server can proxy requests to nodes, pods, and services through its proxy subresources. An identity permitted to use them reaches the kubelet on any node and any ClusterIP service, including internal and unauthenticated ones, using the API server as a pivot that bypasses network segmentation between the attacker and those targets."
keywords:
  - api server proxy
  - nodes proxy
  - services proxy
  - kubelet
  - pivot
---

# API server proxy

The API server exposes proxy subresources that forward a request to a node, a pod, or a service: `nodes/proxy`, `pods/proxy`, and `services/proxy`. Their intended use is cluster introspection, but for an attacker they are a pivot. The API server sits on the cluster network and can reach the kubelet on every node and every ClusterIP service, so proxying through it bypasses whatever segmentation separates the attacker from those targets, and reaches internal services that assume only in-cluster callers can connect.

Check the permission and use the proxy:

```bash
kubectl auth can-i get nodes/proxy
kubectl auth can-i get services/proxy
# proxy to a node's kubelet through the API server
kapi /api/v1/nodes/<node>/proxy/pods
kapi /api/v1/nodes/<node>/proxy/runningpods/
# proxy to an internal ClusterIP service (even if network-isolated from you)
kapi /api/v1/namespaces/<ns>/services/<scheme>:<svc>:<port>/proxy/
```

## Reaching the kubelet for execution

The node proxy fronts the kubelet, so where the kubelet exposes run/exec endpoints, proxying to them runs commands in pods on that node through the API server:

```bash
# proxy an exec/run to the kubelet (effect depends on kubelet config)
kapi -XPOST "/api/v1/nodes/<node>/proxy/run/<ns>/<pod>/<container>" -d 'cmd=id'
```

## Exploitation notes

- `nodes/proxy` is the high-value grant: it fronts the kubelet API on any node, so it combines with the [Kubelet API](kubelet-api.md) routes to list and exec into pods cluster-wide.
- `services/proxy` reaches internal and unauthenticated services regardless of NetworkPolicy between you and them, because the connection originates from the API server; use it to hit dashboards, databases, and internal APIs.
- The proxy preserves the attacker's API identity for authorization at the API server, but the proxied target (a kubelet, an internal service) applies its own, often weaker, authentication.

## References

- [Kubernetes: proxy subresources](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/node-v1/#proxy)
- [Kubernetes: manually constructing apiserver proxy URLs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster-services/)
- [HackTricks: Kubernetes API proxy](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
