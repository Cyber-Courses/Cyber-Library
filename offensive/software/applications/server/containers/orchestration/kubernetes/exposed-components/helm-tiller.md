---
title: "Helm Tiller: cluster control through the legacy Helm v2 server"
description: "Abusing a legacy Helm v2 Tiller service, which runs in-cluster with broad, often cluster-admin permissions and historically no authentication, so any pod that can reach its gRPC port drives it to deploy workloads and read secrets across the cluster."
keywords:
  - Helm Tiller
  - Helm v2
  - 44134
  - tiller service account
  - cluster takeover
---

# Helm Tiller

Helm v2 ran an in-cluster component, Tiller, to apply releases. It typically held a very broad service account (often `cluster-admin`) and, by default, required no authentication on its gRPC port (`44134`). A pod that can reach Tiller inherits its power to install workloads and read secrets everywhere. Helm v3 removed it, but legacy clusters still run it.

```bash
# Tiller's service and port, then drive it with a helm v2 client
kubectl get pods -A | grep tiller
helm --host tiller-deploy.kube-system:44134 ls --all
helm --host tiller-deploy.kube-system:44134 install --name x ./privileged-chart
```

## Exploitation notes

- The payoff is Tiller's service account; where it is `cluster-admin`, reaching Tiller is cluster takeover.
- Installing a crafted chart that creates a privileged, host-mounting pod turns Tiller access into [Pod escape to node](../pod-escape-to-node/index.md).
- Reaching `44134` typically needs an in-cluster foothold, since it is a ClusterIP service.

## References

- [Helm v2 to v3 migration](https://helm.sh/docs/topics/v2_v3_migration/)
- [Helm security](https://helm.sh/docs/topics/securing_installation/)
