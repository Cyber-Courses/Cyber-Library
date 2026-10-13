---
order: 1
title: "QEMU : attaquer l'émulateur de périphériques derrière KVM"
description: "QEMU fournit l'émulation de périphériques pour les machines virtuelles KVM et sous-tend la plupart des produits de virtualisation Linux. Sa surface offensive est l'évasion guest-to-host via les modèles de périphériques, l'interface de contrôle du moniteur QMP/HMP, le vol d'images disque qcow2 et de snapshots, et l'obtention d'un shell sur l'hôte sur lequel QEMU s'exécute. Les bugs du modèle de périphériques se répètent à travers chaque produit basé sur KVM."
keywords:
  - qemu
  - kvm
  - device model
  - qmp
  - qcow2
---

# QEMU

QEMU est l'émulateur de périphériques en espace utilisateur qui s'associe au module noyau KVM pour exécuter des machines virtuelles, et il sous-tend la plupart de la virtualisation Linux : libvirt, Proxmox, OpenStack, oVirt, les plateformes cloud, et plus encore. Sa surface offensive est donc le dénominateur commun des produits basés sur KVM. La préoccupation centrale est l'évasion guest-to-host via les modèles de périphériques de QEMU ; à côté se trouvent le moniteur QMP/HMP qui contrôle une VM en cours d'exécution, les images disque qcow2 et les snapshots qui contiennent les données de l'invité, et l'hôte sur lequel QEMU s'exécute. Parce que chaque produit KVM réutilise ces modèles de périphériques, un bug d'évasion QEMU tend à s'appliquer partout.

```bash
# from the guest: the emulated device inventory (escape surface)
lspci -nn; lsusb; cat /proc/ioports | head
# from the host: QEMU processes and their monitor sockets
ps aux | grep qemu; ls -l /var/lib/libvirt/qemu/*.monitor 2>/dev/null
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape/index.md)** : s'évader via les modèles de périphériques QEMU.
- **[Plan de gestion](management-plane.md)** : le moniteur QMP/HMP et le contrôle libvirt.
- **[Vol de disques et de snapshots](disk-and-snapshot-theft.md)** : voler les images qcow2 et les snapshots.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution sur l'hôte QEMU.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les évasions récurrentes du modèle de périphériques.

## Références

- [Documentation QEMU](https://www.qemu.org/docs/master/)
- [Sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [Recherche Awesome sur l'évasion de VM/hyperviseur](https://github.com/WinMin/Awesome-VM-Exploit)
