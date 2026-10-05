---
title: "Exposed components: reaching unauthenticated Kubernetes services"
description: "Reaching Kubernetes control-plane and node components that are exposed or misconfigured: the API server with anonymous access or a legacy insecure port, the kubelet API, etcd, the dashboard, metrics and cAdvisor, kube-proxy, a legacy Helm Tiller, and recovered kubeconfigs."
keywords:
  - kubernetes exposed components
  - kubelet API
  - etcd
  - anonymous API
  - kubernetes dashboard
---

# Exposed components

A cluster runs many services, and several are dangerous when reachable without authentication. Some are control-plane (the API server, etcd), some are node-local (the kubelet, cAdvisor), and some are add-ons (the dashboard, Tiller). Each can be a direct path to secrets, code execution, or full cluster control.

## Subtopics

- **[Anonymous API access](anonymous-api-access.md)**: the API server accepting unauthenticated requests.
- **[Insecure apiserver port](insecure-apiserver-port.md)**: the legacy unauthenticated port.
- **[Kubelet API](kubelet-api.md)**: the node agent's API, often able to exec in pods.
- **[etcd](etcd.md)**: the cluster datastore, holding every secret.
- **[Dashboard](dashboard.md)**: the web dashboard with a privileged account.
- **[cAdvisor and metrics](cadvisor-and-metrics.md)**: container and node telemetry.
- **[kube-proxy](kube-proxy.md)**: reaching services through an exposed proxy.
- **[Helm Tiller](helm-tiller.md)**: the legacy Helm v2 server with broad rights.
- **[Exposed kubeconfig](exposed-kubeconfig.md)**: recovered admin credentials.

## References

- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [Kubernetes: ports and protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
