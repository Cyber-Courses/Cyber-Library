---
order: 1
title: "Accès à l'hôte et shell : exécution sur le nœud AHV et la Controller VM"
description: "Un déploiement AHV comporte deux shells qu'il vaut la peine d'atteindre : le nœud hyperviseur AHV lui-même et la Controller VM qui exécute la fabrique de stockage et les services de gestion. L'accès provient d'une évasion d'invité, d'identifiants nœud ou CVM, ou de SSH, et la Controller VM est la cible de plus grande valeur car elle médie le stockage et le contrôle du nœud et se connecte au cluster."
keywords:
  - hôte ahv
  - controller vm
  - cvm
  - ssh
  - nutanix
---

# Accès à l'hôte et shell

AHV présente deux cibles de niveau hôte. Le nœud AHV est un hyperviseur Linux exécutant les processus QEMU ; la Controller VM (CVM) est une VM sur ce nœud qui fournit le stockage distribué et héberge les services Nutanix et l'interface Prism. Atteindre un shell sur l'un ou l'autre provient d'une évasion d'invité atterrissant sur le nœud, d'identifiants nœud ou CVM, ou de SSH. La CVM est la cible de plus grande valeur : elle médie toutes les E/S de stockage du nœud, détient les identifiants et clés de service, et est en réseau avec les autres CVM du cluster, donc l'accès CVM se généralise vers le contrôle du cluster.

## Atteindre les shells

```bash
# AHV node (hypervisor) via SSH with node credentials
ssh root@<ahv-node>
virsh list --all                          # VMs on this node, including the CVM
# the CVM, reachable on the internal network from the node (commonly 192.168.5.2)
ssh nutanix@<cvm-ip>
# Nutanix CLIs run on the CVM
acli vm.list; ncli cluster info 2>/dev/null
```

## Ce que donne l'accès CVM

```bash
# the CVM controls storage and management for the node and talks to peer CVMs
# enumerate the cluster and its storage containers (vdisks for every VM)
acli vm.list; acli image.list
ncli storagepool ls; ncli container ls
# service credentials and keys on the CVM pivot to Prism and other CVMs
```

## Notes d'exploitation

- Priorisez la Controller VM plutôt que le nœud nu : elle médie le stockage (elle atteint donc les vdisks de chaque VM), exécute les services de gestion, et est maillée avec les CVM pairs, ce qui en fait le levier sur tout le cluster.
- La CVM se trouve sur un réseau interne nœud-à-CVM (souvent 192.168.5.0/24) accessible depuis le nœud AHV, donc un point d'appui sur le nœud issu d'une évasion d'invité mène naturellement à la CVM.
- Les outils `acli` et `ncli` sur la CVM énumèrent et contrôlent les VM, images et stockage ; utilisez-les pour cartographier le cluster et localiser les VM de grande valeur et leurs disques.
- La persistance sur le nœud ou la CVM suit les mécanismes Linux standard ; c'est la portée en matière de stockage et de gestion qui rend l'accès CVM distinctif.

## Références

- [Nutanix AHV administration](https://portal.nutanix.com/)
- [Nutanix CLI (acli/ncli) reference](https://portal.nutanix.com/)
