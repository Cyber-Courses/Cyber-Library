---
order: 3
title: "Workstation et Fusion : attaquer les hyperviseurs de bureau VMware"
description: "VMware Workstation (Windows/Linux) et Fusion (macOS) sont des hyperviseurs de type 2 qui exécutent les VM en tant que processus vmware-vmx sur l'hôte de l'utilisateur. Ils partagent le code d'émulation de périphériques et de GuestRPC d'ESXi, donc les évasions guest-to-host ciblent les mêmes périphériques, et les fonctionnalités d'intégration de dossiers partagés et de glisser-déposer sont une surface d'évasion proéminente."
keywords:
  - vmware workstation
  - vmware fusion
  - vmware-vmx
  - hyperviseur de type 2
  - dossiers partagés
---

# Workstation et Fusion

Workstation (Windows et Linux) et Fusion (macOS) sont les hyperviseurs hébergés de VMware : chaque VM s'exécute en tant que processus `vmware-vmx` dans la session de l'utilisateur, émulant les mêmes périphériques qu'ESXi. Une évasion d'invité ici corrompt ce processus puis s'exécute sur l'hôte de l'utilisateur. Parce que le code d'émulation de périphériques et de GuestRPC est partagé avec ESXi, la surface d'évasion est la même, SVGA 3D, USB, NIC et les canaux backdoor/VMCI, et les fonctionnalités d'intégration de bureau (dossiers partagés via HGFS, glisser-déposer et copier-coller) ajoutent une surface proéminente et fréquemment vulnérable propre à la manière dont ces produits font le pont entre l'invité et l'hôte.

```bash
# from the guest: same emulated devices as ESXi
lspci -nn; lsusb
# host side: the per-VM process is vmware-vmx (Windows) / vmware-vmx (macOS/Linux)
```

## Sous-sujets

- **[Évasion guest-to-host](guest-to-host-escape.md)** : sortir d'une VM vers l'hôte de l'utilisateur.
- **[Abus des dossiers partagés et du glisser-déposer](shared-folder-and-drag-drop-abuse.md)** : la surface d'intégration HGFS et presse-papiers.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs d'évasion de bureau récurrents.

## Références

- [VMware Workstation documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)
- [Zero Day Initiative: VMware desktop escapes](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
