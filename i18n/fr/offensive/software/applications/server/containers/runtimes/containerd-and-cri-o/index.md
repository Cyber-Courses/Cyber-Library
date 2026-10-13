---
order: 3
title: "containerd et CRI-O : attaquer les runtimes de nœud Kubernetes"
description: "containerd et CRI-O sont les runtimes qui exécutent réellement les conteneurs sous Kubernetes. Leurs sockets de contrôle, s'ils sont accessibles, donnent un contrôle des conteneurs au niveau du nœud équivalent au socket Docker, leurs clients (ctr, crictl) énumèrent tout ce qui se trouve sur le nœud, et les identifiants de pull d'image qu'ils détiennent déverrouillent les registres privés."
keywords:
  - containerd
  - cri-o
  - crictl
  - ctr
  - nœud kubernetes
---

# containerd et CRI-O

Sous Kubernetes, le conteneur qui exécute un pod est créé non par Docker mais par `containerd` ou `CRI-O`, invoqué par le kubelet via la Container Runtime Interface. Leur surface offensive reflète celle de Docker mais est à l'échelle du nœud : un socket de contrôle accessible est équivalent à root sur le nœud, leurs clients énumèrent chaque conteneur, image et task sur le nœud, et les identifiants qu'ils mettent en cache pour pull les images déverrouillent les registres privés.

Détectez quel runtime un nœud utilise et trouvez son socket :

```bash
ls -l /run/containerd/containerd.sock /run/crio/crio.sock /var/run/crio/crio.sock 2>/dev/null
crictl --runtime-endpoint unix:///run/containerd/containerd.sock version 2>/dev/null
ps -ef | grep -E 'containerd|crio' | grep -v grep | head
```

## Sous-sujets

- **[Socket containerd](containerd-socket.md)** : contrôle des conteneurs au niveau du nœud via le socket containerd.
- **[Socket CRI-O](cri-o-socket.md)** : la même chose via le socket CRI-O.
- **[Énumération avec crictl et ctr](crictl-and-ctr-enumeration.md)** : cartographier le nœud avec les clients de runtime.
- **[Vol de secret de pull d'image](image-pull-secret-theft.md)** : dérober les identifiants de registre utilisés par le runtime.

## Références

- [containerd documentation](https://github.com/containerd/containerd/tree/main/docs)
- [CRI-O documentation](https://github.com/cri-o/cri-o/tree/main/docs)
- [Kubernetes: container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
