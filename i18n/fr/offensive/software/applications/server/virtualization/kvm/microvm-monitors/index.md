---
order: 5
title: "Moniteurs microVM : attaquer Firecracker et Cloud Hypervisor"
description: "Firecracker et Cloud Hypervisor sont des moniteurs de périphériques KVM minimaux conçus pour les charges serverless et l'isolation de conteneurs. Ils exposent délibérément un ensemble de périphériques minuscule, virtio sur MMIO, une console série et peu d'autre chose, pour réduire la surface d'évasion. Les attaquer revient à cibler cette surface virtio et API réduite et l'orchestration côté hôte qui gère de nombreuses microVM."
keywords:
  - firecracker
  - cloud hypervisor
  - microvm
  - virtio-mmio
  - modèle de périphériques minimal
---

# Moniteurs microVM

Firecracker (d'AWS) et Cloud Hypervisor sont des Virtual Machine Monitors modernes et minimaux pour KVM, écrits en Rust et conçus pour les usages serverless et d'isolation de conteneurs où la densité et une petite surface d'attaque importent davantage que l'étendue des périphériques. Ils n'émulent délibérément presque rien : une poignée de périphériques virtio sur MMIO (block, net, vsock et quelques autres), une console série et un contrôleur d'interruptions minimal, sans PCI hérité, sans USB, sans graphisme, sans audio. Cela réduit la surface d'évasion guest-to-host aux implémentations virtio et à l'API propre du moniteur, et l'implémentation en Rust élimine la plupart des bugs classiques de sûreté mémoire, donc les attaques se concentrent sur les failles de logique, les blocs `unsafe` et l'orchestration côté hôte.

```bash
# identify the monitor on the host
ps -ef | grep -E 'firecracker|cloud-hypervisor' | grep -v grep
# firecracker is controlled by a REST API on a unix socket
ls -l /run/firecracker*.socket 2>/dev/null
```

## Sous-thèmes

- **[Firecracker](firecracker.md)** : le moniteur microVM d'AWS et sa surface réduite.
- **[Cloud Hypervisor](cloud-hypervisor.md)** : le VMM en Rust et sa surface de périphériques et d'API.

## Références

- [Firecracker design and security](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md)
- [Cloud Hypervisor](https://www.cloudhypervisor.org/)
- [rust-vmm (shared virtio/VMM components)](https://github.com/rust-vmm)
