---
order: 2
title: "Module noyau : attaquer l'interface noyau de KVM"
description: "Le module noyau KVM accélère l'exécution des invités et expose l'interface ioctl /dev/kvm ainsi que l'émulation de périphériques en noyau (comme l'APIC local et le chemin MMIO coalescé). Les bugs ici s'exécutent dans le noyau de l'hôte plutôt que dans un moniteur en espace utilisateur, donc une faille du module KVM est une compromission directe du noyau de l'hôte, et la virtualisation imbriquée élargit la surface d'émulation exposée."
keywords:
  - module kvm
  - /dev/kvm
  - émulation en noyau
  - virtualisation imbriquée
  - noyau de l'hôte
---

# Module noyau

La plupart des évasions KVM ciblent le moniteur en espace utilisateur, mais le module noyau KVM lui-même est une surface, bien plus sévère, car son code s'exécute dans le noyau de l'hôte. KVM accélère la gestion des instructions, de la MMU et des interruptions de l'invité, et émule quelques périphériques en noyau pour la performance (l'APIC local, l'I/O APIC et le PIT, le ring MMIO coalescé, et la gestion MSR/CPUID). Un invité atteint ce code via les opérations que KVM accélère, et une faille de sûreté mémoire ou de logique dans le module est une exécution de code dans le noyau de l'hôte, contournant entièrement tout sandbox de moniteur. La virtualisation imbriquée, où un invité exécute son propre hyperviseur, ajoute les chemins d'émulation VMX/SVM et élargit grandement cette surface.

```bash
# in-kernel emulation and nested support
lsmod | grep -E 'kvm_intel|kvm_amd'
cat /sys/module/kvm_intel/parameters/nested 2>/dev/null   # nested VMX enabled?
cat /sys/module/kvm_amd/parameters/nested 2>/dev/null
```

## Sous-thèmes

- **[Évasion par module noyau](kernel-module-escape.md)** : failles dans l'émulation en noyau accessibles depuis un invité.
- **[Virtualisation imbriquée](nested-virtualization.md)** : la surface d'émulation VMX/SVM qu'expose un invité imbriqué.

## Références

- [KVM API documentation](https://docs.kernel.org/virt/kvm/api.html)
- [Google Project Zero: KVM research](https://googleprojectzero.blogspot.com/)
- [KVM nested virtualization](https://docs.kernel.org/virt/kvm/x86/nested-vmx.html)
