---
title: "Exposed components: attacking reachable cluster control-plane services"
order: 2
description: "Kubernetes clusters run many services that are dangerous when reachable: the API server with anonymous access or an insecure port, the kubelet API on every node, etcd holding all cluster state, the dashboard, cAdvisor and metrics endpoints, leaked kubeconfig files, and legacy Helm Tiller. Each exposed component is a route to secrets, node control, or full cluster compromise."
keywords:
  - kubernetes exposed
  - api server
  - kubelet
  - etcd
  - dashboard
---

# Exposed components

A Kubernetes cluster is many networked services, several of which grant broad control when they are reachable without proper authentication. Some are exposed by misconfiguration (an anonymous-auth API server, an insecure API port, a read-write kubelet), some by design on the node network (cAdvisor, metrics), and some are credentials left where an attacker finds them (a kubeconfig). etcd is the extreme case: it holds the entire cluster state, including every secret, so reaching it is total compromise.

Probe the common exposed surfaces:

```bash
# API server anonymous and insecure port
curl -sk https://<api>:6443/version; curl -s http://<api>:8080/version
# kubelet read-write and read-only ports on a node
curl -sk https://<node>:10250/pods | head; curl -s http://<node>:10255/pods | head
# etcd client port
curl -sk https://<node>:2379/version
# dashboard and metrics
curl -sk https://<node>:30000/ ; curl -s http://<node>:4194/metrics | head
```

## Subtopics

- **[Anonymous API access](anonymous-api-access.md)**: unauthenticated requests the API server accepts.
- **[Insecure apiserver port](insecure-apiserver-port.md)**: the legacy unauthenticated API port.
- **[API server proxy](api-server-proxy.md)**: reaching nodes and services through the API proxy.
- **[Kubelet API](kubelet-api.md)**: the node agent's read-write and read-only APIs.
- **[etcd](etcd.md)**: the datastore holding all cluster secrets.
- **[Dashboard](dashboard.md)**: the web UI and its service account.
- **[cAdvisor and metrics](cadvisor-and-metrics.md)**: container stats endpoints leaking environment data.
- **[Exposed kubeconfig](exposed-kubeconfig.md)**: leaked cluster credentials files.
- **[Helm Tiller](helm-tiller.md)**: the legacy cluster-admin Tiller service.

## References

- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [kube-hunter knowledge base](https://aquasecurity.github.io/kube-hunter/)
- [HackTricks: Kubernetes](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
