---
title: "Dashboard: cluster control through an exposed Kubernetes dashboard"
description: "Abusing an exposed Kubernetes Dashboard, especially older deployments that run with a highly privileged service account and skip login, to view secrets and create workloads through the web UI without any cluster credentials."
keywords:
  - kubernetes dashboard
  - dashboard service account
  - skip login
  - exposed UI
  - cluster takeover
---

# Dashboard

The Kubernetes Dashboard acts with a service account. Older deployments shipped it bound to a very privileged account and allowed skipping login, so anyone who reached the UI inherited that power: reading secrets and creating workloads across the cluster through a browser.

```bash
# Find an exposed dashboard
curl -sk https://<host>/ | grep -i dashboard
# Legacy deployments expose a privileged service account; its token is in a secret
kubectl -n kubernetes-dashboard get secret -o name | grep dashboard
```

## Exploitation notes

- The impact is whatever the dashboard's service account holds; the classic misconfiguration is `cluster-admin` plus skip-login.
- Even with login required, the dashboard's own token (recoverable from its secret via another foothold) grants that account's rights.
- Creating a pod through the dashboard that mounts the host or runs privileged turns UI access into [Pod escape to node](../pod-escape-to-node/index.md).

## References

- [Kubernetes Dashboard: access control](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md)
- [Kubernetes: controlling access to the API](https://kubernetes.io/docs/concepts/security/controlling-access/)
