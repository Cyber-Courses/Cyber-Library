---
order: 3
title: "Évasion guest-to-host : sortir d'une machine virtuelle ESXi"
description: "S'évader d'un invité ESXi revient à corrompre le processus vmx côté hôte qui émule les périphériques de la machine virtuelle. La surface accessible est l'ensemble des périphériques émulés qu'un invité peut piloter : le périphérique graphique SVGA 3D, les contrôleurs USB, les NIC virtuelles et les canaux de contrôle backdoor et VMCI, chacun analysant des données contrôlées par l'invité dans le processus hôte."
keywords:
  - évasion esxi
  - processus vmx
  - émulation de périphériques
  - guest-to-host
  - évasion d'hyperviseur
---

# Évasion guest-to-host

Une machine virtuelle ESXi s'exécute sous un processus `vmx` en espace utilisateur propre à chaque VM sur l'hôte qui émule les périphériques de la VM, soutenu par l'hyperviseur `vmkernel`. Un invité s'évade en faisant manipuler incorrectement par `vmx` des données qu'il contrôle, donc la cible est le code d'émulation de périphériques : le périphérique graphique SVGA avec son flux de commandes 3D, les contrôleurs USB émulés, les cartes réseau virtuelles et les canaux de contrôle backdoor et VMCI. Chacun lit des structures fournies par l'invité (FIFO de commandes, descripteurs de transfert, anneaux, messages RPC) dans le processus hôte, et une faille de sûreté mémoire y donne l'exécution de code dans `vmx`, qui s'exécute avec un privilège suffisant pour atteindre l'hôte.

La surface d'évasion qu'un invité peut atteindre de l'intérieur :

```bash
# enumerate the emulated hardware the VM exposes (from a Linux guest)
lspci -nn                       # SVGA II, vmxnet3/e1000, USB (UHCI/EHCI/XHCI), VMCI
lsusb; ls /sys/bus/pci/devices  # USB controllers and PCI functions
dmesg | grep -iE 'vmwgfx|vmxnet|vmw_vmci'   # loaded VMware guest drivers
```

## Sous-sujets

- **[Backdoor et VMCI](backdoor-and-vmci.md)** : les canaux de contrôle vers le processus vmx.
- **[SVGA et graphismes 3D](svga-and-3d-graphics.md)** : le flux de commandes du périphérique graphique.
- **[Contrôleurs USB](usb-controllers.md)** : les UHCI, EHCI et XHCI émulés.
- **[NIC virtuelles](virtual-nics.md)** : les cartes réseau e1000 et vmxnet3.

## Références

- [Zero Day Initiative: VMware escape research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [Phrack and conference talks on VMware vmx escapes](https://www.zerodayinitiative.com/blog)
