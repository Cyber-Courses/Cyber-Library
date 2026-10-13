---
title: "Composants exposés : attaquer les services accessibles du plan de contrôle du cluster"
order: 2
description: "Les clusters Kubernetes exécutent de nombreux services dangereux lorsqu'ils sont accessibles : l'API server avec accès anonyme ou un port non sécurisé, l'API kubelet sur chaque nœud, etcd qui détient tout l'état du cluster, le dashboard, les endpoints cAdvisor et metrics, les fichiers kubeconfig fuités, et l'ancien Helm Tiller. Chaque composant exposé est une voie vers les secrets, le contrôle des nœuds ou la compromission totale du cluster."
keywords:
  - kubernetes exposed
  - api server
  - kubelet
  - etcd
  - dashboard
---

# Composants exposés

Un cluster Kubernetes est un ensemble de services en réseau, dont plusieurs accordent un contrôle étendu lorsqu'ils sont accessibles sans authentification appropriée. Certains sont exposés par mauvaise configuration (un API server en anonymous-auth, un port d'API non sécurisé, un kubelet en lecture-écriture), d'autres par conception sur le réseau des nœuds (cAdvisor, metrics), et d'autres sont des identifiants laissés à portée d'un attaquant (un kubeconfig). etcd est le cas extrême : il détient l'intégralité de l'état du cluster, y compris chaque secret, de sorte que l'atteindre correspond à une compromission totale.

Sonder les surfaces exposées courantes :

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

## Sous-sujets

- **[Accès anonyme à l'API](anonymous-api-access.md)** : les requêtes non authentifiées que l'API server accepte.
- **[Port apiserver non sécurisé](insecure-apiserver-port.md)** : l'ancien port d'API non authentifié.
- **[Proxy de l'API server](api-server-proxy.md)** : atteindre les nœuds et services via le proxy de l'API.
- **[API kubelet](kubelet-api.md)** : les API en lecture-écriture et en lecture seule de l'agent du nœud.
- **[etcd](etcd.md)** : la base qui détient tous les secrets du cluster.
- **[Dashboard](dashboard.md)** : l'interface web et son compte de service.
- **[cAdvisor et metrics](cadvisor-and-metrics.md)** : les endpoints de statistiques de conteneurs qui fuitent des données d'environnement.
- **[kubeconfig exposé](exposed-kubeconfig.md)** : les fichiers d'identifiants de cluster fuités.
- **[Helm Tiller](helm-tiller.md)** : l'ancien service Tiller en cluster-admin.

## Références

- [Kubernetes : contrôler l'accès à l'API](https://kubernetes.io/docs/concepts/security/controlling-access/)
- [Base de connaissances kube-hunter](https://aquasecurity.github.io/kube-hunter/)
- [HackTricks : Kubernetes](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
