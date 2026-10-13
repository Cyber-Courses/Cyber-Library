---
order: 6
title: "bhyve : attaquer l'hyperviseur FreeBSD"
description: "bhyve est l'hyperviseur de type 2 de FreeBSD, utilisant la virtualisation matérielle avec des modèles de périphériques émulés dans un processus bhyve en espace utilisateur. L'évasion guest-to-host cible ces modèles de périphériques, virtio, AHCI, e1000, ainsi que les périphériques USB et le framebuffer, une faille atterrissant dans le processus bhyve sur l'hôte, aux côtés du vol des images disque des invités."
keywords:
  - bhyve
  - freebsd
  - vmm
  - virtio
  - modèle de périphérique
---

# bhyve

bhyve est l'hyperviseur natif de FreeBSD. Le module noyau `vmm.ko` fournit la virtualisation du CPU et de la mémoire au moyen de VT-x ou AMD-V, tandis qu'un processus `bhyve` en espace utilisateur, propre à chaque VM, émule les périphériques. La conception est épurée et moderne, centrée sur virtio et un petit ensemble de périphériques émulés (stockage AHCI, carte réseau e1000, USB, un framebuffer et le passthrough PCI). Une évasion guest-to-host cible donc les modèles de périphériques du processus `bhyve`, atterrissant sur l'hôte FreeBSD, et le vol habituel des images disque s'applique au stockage sous-jacent de l'invité.

```bash
# on a FreeBSD bhyve host
kldstat | grep vmm; ls /dev/vmm/
ps aux | grep bhyve | grep -v grep
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape.md)** : sortir via les modèles de périphériques de bhyve.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution sur l'hôte FreeBSD.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs récurrents des modèles de périphériques.

## Références

- [FreeBSD Handbook: bhyve](https://docs.freebsd.org/en/books/handbook/virtualization/#virtualization-host-bhyve)
- [bhyve(8) manual](https://man.freebsd.org/cgi/man.cgi?query=bhyve)
- [FreeBSD security advisories](https://www.freebsd.org/security/advisories/)
