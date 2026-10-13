---
order: 2
title: "Évasion guest-to-host : sortir d'une machine virtuelle QEMU/KVM"
description: "Un invité QEMU s'évade en corrompant le processus QEMU sur l'hôte, qui émule les périphériques de la VM tandis que KVM accélère le CPU. La surface accessible est le modèle de périphériques de QEMU : périphériques virtio, NIC émulées, contrôleurs USB, contrôleurs bloc et SCSI, périphériques audio, et le contrôleur de disquette legacy, chacun analysant des écritures de registre et des descripteurs DMA pilotés par l'invité."
keywords:
  - qemu escape
  - kvm
  - device model
  - virtio
  - guest-to-host
---

# Évasion guest-to-host

QEMU fournit l'émulation de périphériques pour une machine virtuelle KVM tandis que le module noyau KVM accélère la virtualisation du CPU et de la mémoire. Un invité ne peut pas toucher au chemin CPU de l'hôte, si bien que les évasions ciblent le processus en espace utilisateur QEMU : ses modèles de périphériques lisent les E/S pilotées par l'invité (écritures de registre port et MMIO) et parcourent les descripteurs DMA et les ring buffers que l'invité place dans sa propre mémoire. Une faille de sûreté mémoire dans n'importe quel modèle de périphérique donne l'exécution de code dans le processus QEMU, qui s'exécute sur l'hôte avec les privilèges de celui qui a lancé la VM (souvent réduits par seccomp et parfois confinés, mais tout de même un processus hôte). Parce que les modèles de périphériques sont partagés, ces bugs se répètent à travers chaque produit basé sur KVM.

```bash
# from the guest: enumerate the emulated devices (the escape surface)
lspci -nn                       # virtio, e1000/rtl8139, USB controllers, audio
lsusb; cat /proc/ioports        # I/O-port devices incl. the legacy floppy (0x3f0-)
dmesg | grep -i virtio
```

## Sous-thèmes

- **[périphériques virtio](virtio-devices.md)** : la famille de périphériques paravirtuels à virtqueue.
- **[Adaptateurs réseau](network-adapters.md)** : e1000, rtl8139 et virtio-net émulés.
- **[Contrôleurs USB](usb-controllers.md)** : UHCI, EHCI et XHCI émulés.
- **[Bloc et SCSI](block-and-scsi.md)** : les chemins de stockage IDE/AHCI et virtio-blk/SCSI.
- **[Périphériques audio](audio-devices.md)** : les cartes son émulées.
- **[Contrôleur de disquette](floppy-controller.md)** : le contrôleur de disquette legacy.

## Références

- [Documentation QEMU](https://www.qemu.org/docs/master/)
- [Processus de sécurité et avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [Recherche Awesome sur l'évasion de VM/hyperviseur](https://github.com/WinMin/Awesome-VM-Exploit)
