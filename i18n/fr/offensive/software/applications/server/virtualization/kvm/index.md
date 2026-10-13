---
order: 7
title: "KVM : attaquer l'hyperviseur du noyau Linux et ses produits"
description: "KVM transforme le noyau Linux en hyperviseur, accélérant les VM tandis qu'un moniteur en espace utilisateur (QEMU, Firecracker, Cloud Hypervisor) émule les périphériques. La surface d'attaque couvre le module noyau KVM lui-même, les moniteurs émulant les périphériques, et les produits bâtis sur KVM, plateformes basées sur QEMU, Proxmox VE et Nutanix AHV, chacun ajoutant son propre plan de gestion."
keywords:
  - kvm
  - qemu
  - firecracker
  - proxmox
  - nutanix ahv
---

# KVM

KVM (Kernel-based Virtual Machine) est le moteur de virtualisation du noyau Linux : il utilise la virtualisation matérielle pour exécuter efficacement les CPU invités tandis qu'un Virtual Machine Monitor en espace utilisateur émule les périphériques. Cette séparation définit la surface d'attaque. Le module noyau KVM expose une interface (les ioctls `/dev/kvm` et l'accélération en noyau des instructions, de la MMU et de certains périphériques) qui est elle-même une surface d'évasion et d'élévation de privilèges. Les moniteurs, QEMU, les moniteurs microVM Firecracker et Cloud Hypervisor, fournissent les modèles de périphériques par lesquels un invité s'évade. Et les produits bâtis sur KVM, plateformes basées sur QEMU, Proxmox VE et Nutanix AHV, l'enveloppent chacun d'un plan de gestion qui constitue sa propre cible.

```bash
# host: KVM in use and which monitor
ls -l /dev/kvm; lsmod | grep kvm
ps -ef | grep -E 'qemu|firecracker|cloud-hypervisor' | grep -v grep
```

## Sous-thèmes

- **[QEMU](qemu/index.md)** : l'émulateur de périphériques dominant derrière KVM.
- **[Module noyau](kernel-module/index.md)** : l'interface noyau KVM et la virtualisation imbriquée.
- **[Moniteurs microVM](microvm-monitors/index.md)** : Firecracker et Cloud Hypervisor.
- **[Proxmox VE](proxmox-ve/index.md)** : la plateforme de virtualisation basée sur KVM.
- **[Nutanix AHV](nutanix-ahv/index.md)** : l'hyperviseur hyperconvergé basé sur KVM.

## Références

- [KVM documentation (kernel)](https://docs.kernel.org/virt/kvm/index.html)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM/hypervisor escape](https://github.com/WinMin/Awesome-VM-Exploit)
