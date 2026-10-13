---
order: 1
title: "Accès à l'hôte et shell : exécution sur le nœud Proxmox"
description: "Un nœud Proxmox est un hôte Debian exécutant les VM, les conteneurs et les daemons de gestion en root. L'accès provient d'une évasion de VM ou de conteneur, de SSH, ou de l'API de gestion et du shell web. Avec lui, un attaquant contrôle chaque VM et conteneur via qm/pct, lit tous les disques et sauvegardes, et dans un cluster atteint les autres nœuds via la configuration partagée adossée à corosync."
keywords:
  - proxmox node
  - pvedaemon
  - qm pct
  - corosync
  - cluster
---

# Accès à l'hôte et shell

Un nœud Proxmox est un serveur Debian où les daemons de gestion (`pvedaemon`, `pveproxy`, `pve-cluster`) et les processus VM/conteneur s'exécutent en root. L'obtention d'un shell provient d'une évasion de VM ou de conteneur atterrissant sur le nœud, de SSH avec des identifiants de nœud, ou de l'interface de gestion, qui inclut un shell web intégré (noVNC/xterm.js) et l'exécution de commandes via l'API. Le root du nœud équivaut au contrôle de chaque VM et conteneur présent via `qm` et `pct`, à l'accès à tous les disques et sauvegardes, et, dans un cluster, à la portée vers les autres nœuds car le système de fichiers du cluster et corosync les relient.

## Atteindre et utiliser le nœud

```bash
# SSH with node credentials, or the web-UI shell
ssh root@<proxmox-node>
qm list; pct list                         # VMs and containers
# run inside a VM via the guest agent, or in a container directly
qm guest exec <vmid> -- id 2>/dev/null
pct exec <ctid> -- id
```

## Portée vers le cluster

```bash
# the cluster filesystem (pmxcfs) is shared across nodes under /etc/pve
pvecm status; pvecm nodes
cat /etc/pve/corosync.conf                 # cluster members
ls /etc/pve/nodes/                         # per-node config, shared cluster-wide
# node root on one member, plus the shared /etc/pve and SSH trust, reaches peers
ssh root@<other-node>                      # cluster nodes commonly trust each other
```

`/etc/pve` est le système de fichiers du cluster adossé à corosync (pmxcfs) répliqué sur chaque nœud, et les membres du cluster partagent généralement une confiance SSH root, si bien que compromettre un nœud s'étend couramment à tout le cluster.

## Notes d'exploitation

- Le root du nœud englobe tout : `qm`/`pct` contrôlent tous les invités, le guest agent exécute des commandes dans les VM, et `pct exec` entre directement dans les conteneurs.
- Le système de fichiers de cluster partagé sous `/etc/pve` et la confiance SSH inter-nœuds habituelle font d'une compromission d'un seul nœud une compromission du cluster ; énumérez les pairs via `pvecm nodes` et la config partagée.
- Le shell de l'interface web et l'exécution de commandes de l'API sont des voies d'accès légitimes au nœud avec des identifiants de gestion, une alternative à une évasion depuis l'invité.
- La persistance est du Debian standard (systemd, cron, clés SSH) ; le système de fichiers du cluster offre un emplacement répliqué visible par tous les nœuds.

## Références

- [Proxmox VE : gestionnaire de cluster (pmxcfs/corosync)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html)
- [Administration de Proxmox VE](https://pve.proxmox.com/pve-docs/)
