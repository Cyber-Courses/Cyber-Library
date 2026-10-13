---
order: 9
title: "Virtualisation : attaquer les hyperviseurs et leurs plans de gestion"
description: "Attaquer les plateformes de virtualisation : s'évader d'une VM invitée vers l'hôte via des failles de l'hyperviseur et de l'émulation de périphériques, prendre le contrôle du plan de gestion qui pilote de nombreux hôtes, et voler les disques virtuels et snapshots hors ligne, à travers VMware, Hyper-V, KVM, Xen, Proxmox, Nutanix, et d'autres."
keywords:
  - sécurité de la virtualisation
  - hyperviseur
  - évasion de VM
  - évasion guest-to-host
  - plan de gestion
---

# Virtualisation

Un hyperviseur exécute des systèmes d'exploitation invités non fiables et est censé les maintenir isolés de l'hôte et les uns des autres. Le travail offensif cible les endroits où cette isolation fuit : les modèles de périphériques que l'hyperviseur émule pour les invités (la surface d'évasion guest-to-host habituelle), le plan de gestion qui pilote des flottes d'hôtes, et les disques virtuels et snapshots qui contiennent les données des invités au repos.

Le domaine est organisé par produit, car les évasions sont spécifiques au code d'émulation de périphériques de chaque hyperviseur, avec un ensemble cohérent d'aspects sous chacun.

## Dans quel hyperviseur suis-je

Depuis un invité, identifiez l'hyperviseur avant de choisir une page produit ; les périphériques émulés et les signatures le trahissent :

```bash
# CPUID hypervisor vendor and DMI/BIOS strings are the quickest tells
systemd-detect-virt 2>/dev/null                 # kvm/vmware/microsoft/xen/oracle/parallels/bhyve
dmidecode -s system-product-name 2>/dev/null    # "VMware Virtual Platform", "VirtualBox", "KVM"...
lscpu | grep -i hypervisor; grep -o hypervisor /proc/cpuinfo | head -1
# emulated devices confirm it
lspci -nn                                        # virtio => KVM/bhyve; vmxnet/SVGA => VMware;
                                                 # Hyper-V VMBus; Parallels/VirtualBox PCI IDs
dmesg | grep -iE 'vmware|hyper-v|kvm|xen|virtualbox|parallels|bhyve'
```

Les trois aspects d'attaque reviennent sous chaque produit : l'**évasion guest-to-host** via les périphériques émulés (nécessite un exploit, spécifique au modèle de périphérique), le **plan de gestion** qui pilote de nombreux hôtes (nécessite souvent seulement des identifiants ou une API accessible), et le **vol de disques et de snapshots** qui lit les données des invités hors ligne (nécessite un accès hôte, stockage ou gestion, sans exploit d'invité). Choisissez l'aspect qui correspond à l'accès dont vous disposez déjà.

## Sous-thèmes

- **[VMware](vmware/index.md)** : ESXi et vCenter, plus les hyperviseurs de bureau Workstation et Fusion.
- **[Hyper-V](hyper-v/index.md)** : l'hyperviseur Windows et sa pile de gestion.
- **[VirtualBox](virtualbox/index.md)** : l'hyperviseur de bureau d'Oracle.
- **[KVM](kvm/index.md)** : l'écosystème KVM de Linux : QEMU, le module noyau, Proxmox VE, Nutanix AHV, et les moniteurs microVM Firecracker et Cloud Hypervisor.
- **[Xen](xen/index.md)** : l'hyperviseur Xen et son domaine de contrôle.
- **[Parallels Desktop](parallels-desktop/index.md)** : l'hyperviseur de bureau macOS.
- **[bhyve](bhyve/index.md)** : l'hyperviseur FreeBSD.

## Références

- [Pwn2Own results archive](https://www.zerodayinitiative.com/blog)
- [QEMU security process](https://www.qemu.org/docs/master/system/security.html)
