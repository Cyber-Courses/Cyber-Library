---
title: "Évasion par module noyau : compromission du noyau de l'hôte via l'émulation en noyau de KVM"
description: "KVM émule certains périphériques et instructions à l'intérieur du noyau de l'hôte pour la performance : l'APIC local, l'I/O APIC, le PIT, le MMIO coalescé et la gestion MSR/CPUID. Un invité atteint ces chemins via MMIO, les E/S et les instructions privilégiées, de sorte qu'une faille de sûreté mémoire ou de logique là s'exécute directement dans le noyau de l'hôte, une évasion plus puissante que la corruption d'un moniteur en espace utilisateur."
keywords:
  - évasion kvm
  - émulation en noyau
  - apic local
  - mmio coalescé
  - noyau de l'hôte
---

# Évasion

Le module KVM émule un sous-ensemble critique pour la performance de la plateforme dans le noyau de l'hôte, et ce code est accessible depuis l'invité. L'APIC local et l'I/O APIC en noyau, le timer PIT, le ring-buffer MMIO coalescé, et la gestion de certains MSR, de CPUID et l'émulation d'instructions traitent tous des événements déclenchés par l'invité en contexte noyau. Une faille là, un accès hors limites dans la gestion des registres de l'APIC, un mauvais index dans le ring MMIO coalescé, une erreur de logique dans l'émulation d'instructions, est une exécution de code ou une corruption mémoire dans le noyau de l'hôte, strictement plus puissante qu'une évasion d'un moniteur en espace utilisateur car elle saute le sandbox du moniteur et atterrit en ring 0 sur l'hôte.

## La surface en noyau

```c
// a guest reaches in-kernel KVM code by the operations KVM handles without exiting
// to the monitor:
//  - APIC: MMIO/x2APIC MSR accesses to the emulated local APIC register page
//  - coalesced MMIO: writes to registered regions fill an in-kernel ring the guest
//    and host share; a bad first/last index is a classic OOB primitive
//  - instruction emulation: KVM's x86 emulator decodes and executes some guest
//    instructions in-kernel; operand/memory handling bugs are reachable here
//  - MSR/CPUID: handlers for guest reads/writes of model-specific registers
```

Les endroits historiquement productifs sont la gestion de l'index du ring MMIO coalescé et l'émulateur d'instructions x86, qui traitent tous deux une entrée riche et contrôlée par l'invité dans le noyau.

## Notes d'exploitation

- Atterrir dans le noyau de l'hôte est le trait distinctif : contrairement à une évasion QEMU, il n'y a pas de sandbox en espace utilisateur à ensuite franchir, donc un bug du module KVM est une compromission directe de l'hôte en ring 0.
- Le ring MMIO coalescé est une structure partagée invité/hôte avec des index que l'invité peut influencer, faisant des bugs de bornage d'index une source OOB récurrente ; l'émulateur d'instructions est un décodeur complexe accessible pour certaines instructions.
- Atteindre ces chemins nécessite que l'invité effectue le MMIO/MSR/instruction déclencheur, ce qu'un noyau invité contrôlé par l'attaquant fait directement.
- Spécifique à la version du noyau en cours d'exécution ; identifiez `uname -r` et le module kvm, et notez que la virtualisation imbriquée élargit cette surface, voir [Virtualisation imbriquée](nested-virtualization.md).

## Références

- [KVM API and in-kernel devices](https://docs.kernel.org/virt/kvm/api.html)
- [Google Project Zero: KVM in-kernel emulation research](https://googleprojectzero.blogspot.com/)
- [Awesome VM escape: KVM](https://github.com/WinMin/Awesome-VM-Exploit)
