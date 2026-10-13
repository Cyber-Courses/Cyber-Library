---
order: 4
title: "Nutanix AHV : attaquer l'hyperviseur KVM hyperconvergé"
description: "Nutanix AHV est un hyperviseur basé sur KVM/QEMU dans une plateforme hyperconvergée où une Controller VM sur chaque nœud exécute la fabrique de stockage et les services de gestion. L'évasion guest-to-host hérite des modèles de périphériques de QEMU ; la surface distinctive est la Controller VM, le plan de gestion Prism et le stockage distribué contenant les disques virtuels de chaque VM."
keywords:
  - nutanix ahv
  - controller vm
  - prism
  - kvm
  - hyperconvergé
---

# Nutanix AHV

Nutanix AHV est l'hyperviseur natif de l'entreprise, bâti sur KVM avec une émulation de périphériques dérivée de QEMU. Ce qui le distingue est l'architecture hyperconvergée : chaque nœud exécute une Controller VM (CVM) qui fournit la fabrique de stockage distribuée et héberge les services de gestion, avec l'interface Prism (Element par cluster et Central multi-cluster) au-dessus. Offensivement, l'évasion guest-to-host est la surface de périphériques QEMU familière, mais les cibles de grande valeur sont la Controller VM, qui médie le stockage et la gestion, l'API Prism, et le stockage distribué qui contient les disques virtuels de chaque VM.

```bash
# on an AHV host: KVM/QEMU plus the Nutanix stack
ps -ef | grep -E 'qemu|acropolis' | grep -v grep
# the Controller VM runs the storage and management services
virsh list --all 2>/dev/null | grep -i cvm
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape.md)** : sortir d'une VM AHV via les modèles de périphériques QEMU.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution sur l'hôte AHV et la Controller VM.
- **[Plan de gestion](management-plane.md)** : l'API Prism et les services de la Controller VM.
- **[Vol de disques et de snapshots](disk-and-snapshot-theft.md)** : récupérer les vdisks depuis la fabrique de stockage.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs hérités de QEMU et spécifiques à Nutanix.

## Références

- [Nutanix AHV documentation](https://www.nutanix.com/products/ahv)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
- [QEMU security](https://www.qemu.org/docs/master/system/security.html)
