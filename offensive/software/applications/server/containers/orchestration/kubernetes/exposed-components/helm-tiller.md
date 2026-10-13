---
title: "Helm Tiller: cluster-admin through the legacy Helm server component"
order: 8
description: "Helm v2 ran a cluster-side component, Tiller, usually with a cluster-admin service account and a gRPC API on port 44134 that performed no authentication. A pod that can reach Tiller submits chart operations that Tiller executes with its own cluster-admin rights, so reaching Tiller is full cluster control through a component unrelated to the attacker's own permissions."
keywords:
  - helm tiller
  - port 44134
  - helm v2
  - cluster-admin
  - unauthenticated
---

# Helm Tiller

Helm version 2 installed a server-side component called Tiller into the cluster to apply charts. Tiller ran with a service account that was, by the common install instructions, bound to `cluster-admin`, and it exposed a gRPC API (port 44134) that performed no authentication: any client that could reach it could ask it to install or modify releases, which Tiller executed with its cluster-admin rights. A pod able to reach Tiller therefore inherits cluster-admin regardless of its own permissions. Helm 3 removed Tiller, but clusters still running Helm 2, or with a leftover Tiller, remain exposed.

Find Tiller:

```bash
kubectl get pods -A | grep -i tiller
kubectl get svc -A | grep -i tiller                   # tiller-deploy, port 44134
# reachability from a pod
nc -z -w1 tiller-deploy.kube-system 44134 && echo reachable
```

## Abuse Tiller's privileges

```bash
# point a helm v2 client at the in-cluster Tiller and act through it
export HELM_HOST=tiller-deploy.kube-system:44134
helm2 ls                                               # lists releases via Tiller
# install a chart that creates a cluster-admin binding for the attacker,
# or a privileged pod; Tiller applies it as cluster-admin
helm2 install --name x ./evil-chart
```

A minimal malicious chart ships a ClusterRoleBinding granting the attacker's service account `cluster-admin`, or a privileged pod; Tiller creates it with its own rights, so the normal anti-escalation checks do not apply to the attacker.

## Exploitation notes

- The exposure is that Tiller, not the caller, holds cluster-admin and authenticates no one, so the only gate is network reachability to port 44134, commonly open to all pods.
- Ship a chart whose templates create a durable grant (a ClusterRoleBinding to `cluster-admin` for your service account); this persists after Tiller is removed, unlike a one-off action.
- This applies only to Helm 2 era clusters; confirm a Tiller pod or `tiller-deploy` service exists before pursuing it.

## Tools

- [helm v2 client](https://github.com/helm/helm/releases)

## References

- [Helm v2 Tiller and security](https://v2.helm.sh/docs/securing_installation/)
- [HackTricks: Helm Tiller](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
