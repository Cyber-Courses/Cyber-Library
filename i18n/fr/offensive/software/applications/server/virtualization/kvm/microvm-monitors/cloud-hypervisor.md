---
order: 2
title: "Cloud Hypervisor : attaquer le VMM en Rust"
description: "Cloud Hypervisor est un moniteur KVM en Rust doté d'un ensemble de périphériques moderne mais encore limité, périphériques virtio, PCI et passthrough optionnels, et une API de contrôle HTTP. La surface guest-to-host est sa gestion de virtio et VFIO et les composants rust-vmm partagés ; la surface hôte est l'API et la configuration du passthrough et du sandboxing."
keywords:
  - cloud hypervisor
  - rust-vmm
  - virtio
  - vfio
  - api http
---

# Cloud Hypervisor

Cloud Hypervisor est un moniteur KVM basé sur Rust visant les charges de travail modernes de cloud et de conteneurs. Il offre davantage que Firecracker, périphériques virtio, PCI optionnel et passthrough de périphériques VFIO, tout en omettant encore la plupart de l'émulation héritée, et partage des briques virtio et VMM avec le projet `rust-vmm`. La surface d'évasion guest-to-host est sa gestion des périphériques virtio, le chemin de passthrough VFIO lorsqu'il est activé, et les composants rust-vmm partagés ; la surface côté hôte est l'API de contrôle HTTP et les choix de configuration (passthrough, sandboxing) qui élargissent ou restreignent l'exposition.

## Les surfaces

```bash
# guest: the virtio (and optionally passthrough PCI) devices
ls /sys/bus/virtio/devices 2>/dev/null; lspci 2>/dev/null
# host: the HTTP API socket controls the VM lifecycle and device config
curl --unix-socket /run/cloud-hypervisor.sock -s http://localhost/api/v1/vm.info
```

```text
Escape and abuse surface:
- virtio device virtqueue handling (shared rust-vmm/virtio code) - the main guest surface
- VFIO device passthrough, when configured: a passed-through physical device widens
  the surface to that device's driver and DMA, and weakens isolation versus emulation
- the HTTP API: lifecycle and device (disk/net) configuration, i.e. VM control
- shared rust-vmm components mean a bug there can affect multiple VMMs built on them
```

## Notes d'exploitation

- La gestion des virtqueues virtio est la surface guest-to-host, implémentée en Rust via le code rust-vmm partagé, donc les failles de logique et l'usage de `unsafe` sont les classes de bugs réalistes ; voir [périphériques virtio](../qemu/guest-to-host-escape/virtio-devices.md) pour le mécanisme.
- Le passthrough VFIO, lorsqu'il est activé, est une configuration à double tranchant : il donne à une VM un accès direct à un périphérique physique, agrandissant la surface à ce périphérique et à son DMA et réduisant l'isolation que procure l'émulation pure.
- L'API HTTP est le contrôle de la VM : atteindre son socket permet de reconfigurer les périphériques et le cycle de vie de la VM, donc son contrôle d'accès importe autant que le code des périphériques.
- Les composants rust-vmm partagés créent une pertinence inter-VMM : une faille dans un crate virtio partagé peut s'appliquer à d'autres moniteurs bâtis sur rust-vmm.

## Références

- [Cloud Hypervisor documentation](https://www.cloudhypervisor.org/docs/)
- [rust-vmm project](https://github.com/rust-vmm)
- [Cloud Hypervisor API](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/main/docs/api.md)
