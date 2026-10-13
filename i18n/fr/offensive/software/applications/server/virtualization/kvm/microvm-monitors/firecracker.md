---
order: 1
title: "Firecracker : attaquer le moniteur microVM d'AWS"
description: "Firecracker n'émule que virtio-block, virtio-net, virtio-vsock, une console série et un ensemble minimal de contrôleurs, et il est écrit en Rust avec un jailer seccomp. La surface guest-to-host est constituée de ces périphériques virtio et de leur gestion de virtqueues ; le côté hôte ajoute l'API de contrôle REST et la configuration du jailer, où une mauvaise configuration ou une faille de logique affaiblit l'isolation voulue."
keywords:
  - firecracker
  - virtio
  - jailer
  - seccomp
  - api rest
---

# Firecracker

Firecracker exécute chaque microVM avec un ensemble de périphériques minuscule : virtio-block, virtio-net, virtio-vsock, une console série, un contrôleur d'interruptions minimal et quelques périphériques de plateforme, rien d'autre. Il est écrit en Rust et livré avec un `jailer` qui enveloppe chaque instance d'un filtre seccomp, d'un chroot et de namespaces et cgroups dédiés. La surface d'évasion guest-to-host est donc étroite : les périphériques virtio et leur gestion de virtqueues. La surface côté hôte est l'API de contrôle REST (sur un socket Unix) et le confinement du jailer, où une configuration faible ou une faille de logique dans la gestion des périphériques ou de l'API est la cible réaliste plutôt qu'un bug classique de corruption mémoire.

## Les surfaces

```bash
# guest side: the only paravirtual devices present
lspci 2>/dev/null; ls /sys/bus/virtio/devices 2>/dev/null   # virtio-mmio block/net/vsock
# host side: the control API socket (per microVM)
curl --unix-socket /run/firecracker.socket -s http://localhost/machine-config
```

```text
Escape and abuse surface:
- virtio-block / -net / -vsock virtqueue descriptor handling (the only device parsing)
- the vsock device, which bridges guest and host sockets, is a notable logic surface
- the REST API: whoever reaches the control socket can reconfigure/boot/stop the VM,
  attach drives (host files), and set up networking - control of the microVM
- the jailer: if the seccomp profile, chroot, or namespaces are weakened or bypassed,
  a device-layer bug that would be contained becomes a broader host compromise
```

## Notes d'exploitation

- La gestion des virtqueues virtio est la surface guest-to-host ; les mécanismes correspondent aux [périphériques virtio](../qemu/guest-to-host-escape/virtio-devices.md), mais l'implémentation est en Rust, donc les bugs réalistes sont des erreurs de logique et des blocs `unsafe` plutôt que des débordements omniprésents.
- vsock est une surface distinctive car il relie les sockets de l'invité aux sockets de l'hôte ; sa gestion des connexions et des buffers est une cible à failles de logique.
- Le socket de l'API REST équivaut au contrôle de la microVM : l'atteindre permet à un attaquant d'attacher des fichiers de l'hôte comme disques ou de reconfigurer la VM, donc son contrôle d'accès (permissions du socket, jailer) est critique.
- Le confinement seccomp, chroot et namespaces du jailer est la couche de défense en profondeur ; sa robustesse détermine si une compromission au niveau périphérique reste contenue, donc évaluez la configuration du jailer.

## Références

- [Firecracker design](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Firecracker jailer](https://github.com/firecracker-microvm/firecracker/blob/main/docs/jailer.md)
- [Firecracker API](https://github.com/firecracker-microvm/firecracker/blob/main/src/firecracker/swagger/firecracker.yaml)
