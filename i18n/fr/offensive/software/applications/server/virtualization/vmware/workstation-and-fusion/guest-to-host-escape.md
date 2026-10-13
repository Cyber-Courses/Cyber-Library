---
order: 1
title: "Évasion guest-to-host : sortir de Workstation et Fusion vers l'hôte"
description: "Un invité Workstation ou Fusion s'évade en corrompant le processus vmware-vmx qui émule ses périphériques, exactement comme sur ESXi mais en atterrissant sur l'hôte de bureau de l'utilisateur. Les mêmes surfaces SVGA 3D, USB, NIC et GuestRPC/VMCI s'appliquent, et l'exécution de code dans vmware-vmx s'exécute avec les privilèges de l'utilisateur ou du service exécutant la VM."
keywords:
  - vmware-vmx
  - guest-to-host
  - svga
  - guestrpc
  - évasion de bureau
---

# Évasion guest-to-host

Sur Workstation et Fusion, chaque VM est émulée par un processus `vmware-vmx` dans la session de l'utilisateur hôte, de sorte qu'une évasion guest-to-host est le même problème de corruption mémoire que sur ESXi, avec le gain atterrissant sur l'hôte de bureau. Les surfaces accessibles sont identiques car le code est partagé : le périphérique SVGA II et son flux de commandes 3D, les contrôleurs USB émulés, les NIC virtuelles, et les canaux de contrôle backdoor GuestRPC et VMCI. Une faille de sûreté mémoire dans l'un d'eux donne l'exécution de code dans `vmware-vmx`, qui s'exécute avec les privilèges de celui qui a démarré la VM.

```bash
# the escape surface, enumerated from inside the guest (same as ESXi)
lspci -nn | grep -iE 'svga|vmxnet|usb|vmci'
lsusb; dmesg | grep -iE 'vmwgfx|vmxnet|vmw_vmci'
```

Les mécanismes par périphérique sont documentés sous les pages guest-to-host d'ESXi, puisque les modèles de périphériques sont partagés :

- Analyse du flux de commandes SVGA 3D, voir [SVGA et graphismes 3D](../esxi/guest-to-host-escape/svga-and-3d-graphics.md).
- Gestion des descripteurs et des anneaux des contrôleurs USB, voir [Contrôleurs USB](../esxi/guest-to-host-escape/usb-controllers.md).
- Gestion des anneaux et des descripteurs des NIC virtuelles, voir [NIC virtuelles](../esxi/guest-to-host-escape/virtual-nics.md).
- Les canaux backdoor GuestRPC et VMCI, voir [Backdoor et VMCI](../esxi/guest-to-host-escape/backdoor-and-vmci.md).

## Ce qui diffère d'ESXi

```bash
# the host is a desktop OS, so post-escape you are in the user's session:
#  - Windows: vmware-vmx.exe runs as the logged-in user (or elevated if the VM was)
#  - macOS/Linux: vmware-vmx runs as the user; Fusion uses a helper for some ops
# privilege after escape = the VM-launching user's, then local privesc on the host
```

## Notes d'exploitation

- Les classes de bugs des modèles de périphériques sont partagées avec ESXi, donc les mécanismes et l'énumération sont identiques ; la différence est l'environnement d'atterrissage, une session utilisateur de bureau plutôt que l'hôte vmkernel.
- Le privilège post-évasion égale celui de l'utilisateur exécutant la VM ; une élévation de privilèges locale supplémentaire sur l'OS de bureau est souvent nécessaire pour le contrôle complet de l'hôte, contrairement à ESXi où `vmx` est déjà fortement privilégié.
- Les fonctionnalités d'intégration (dossiers partagés, glisser-déposer) sont plus couramment activées sur les produits de bureau que sur les serveurs, rendant la surface GuestRPC/HGFS particulièrement pertinente ici ; voir [Abus des dossiers partagés et du glisser-déposer](shared-folder-and-drag-drop-abuse.md).

## Références

- [Zero Day Initiative: VMware Workstation/Fusion escapes](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
