---
order: 3
title: "Proxmox VE : attaquer la plateforme de virtualisation KVM et LXC"
description: "Proxmox VE est une plateforme basée sur Debian exécutant des machines virtuelles KVM/QEMU et des conteneurs LXC, administrée via une interface web et une API REST sur le port 8006 et mise en cluster avec corosync. L'évasion guest-to-host hérite des modèles de périphériques QEMU ; la surface distinctive est l'API de gestion et l'interface web, le cluster, et les backends de stockage qui contiennent les disques de VM."
keywords:
  - proxmox ve
  - qemu
  - lxc
  - pvedaemon
  - corosync
---

# Proxmox VE

Proxmox VE est une plateforme de virtualisation basée sur Debian qui exécute à la fois des machines virtuelles KVM/QEMU et des conteneurs LXC, administrée via une interface web et une API REST (les services `pvedaemon`/`pveproxy` sur le port 8006) et mise en cluster avec corosync. Sa surface d'attaque comporte trois parties. L'évasion guest-to-host depuis une VM Proxmox est le problème du modèle de périphériques QEMU, hérité en bloc. Le plan de gestion, l'API, l'interface web, et l'outillage `pvesh`/`qm`/`pct`, contrôle chaque VM et conteneur. Et les backends de stockage (répertoires locaux, LVM, ZFS, Ceph) contiennent les disques de VM et les sauvegardes. Les conteneurs LXC ajoutent la surface d'évasion de conteneur à côté de celle des VM.

```bash
# on a Proxmox node
qm list; pct list                         # KVM VMs and LXC containers
ss -tlnp | grep 8006                       # the management web/API
pvecm status 2>/dev/null                   # cluster membership
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape.md)** : sortir d'une VM Proxmox via QEMU.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution sur le nœud Proxmox.
- **[Plan de gestion](management-plane.md)** : l'API, l'interface web et le cluster.
- **[Vol de disques et de sauvegardes](disk-and-backup-theft.md)** : récupérer les disques de VM et les sauvegardes vzdump.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs QEMU hérités et ceux de la plateforme.

## Références

- [Documentation Proxmox VE](https://pve.proxmox.com/pve-docs/)
- [Visionneuse d'API Proxmox VE](https://pve.proxmox.com/pve-docs/api-viewer/)
- [Sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
