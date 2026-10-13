---
title: "Virtualisation imbriquée : la surface d'émulation VMX et SVM"
description: "La virtualisation imbriquée permet à un invité KVM d'exécuter son propre hyperviseur, ce qui exige de KVM qu'il émule les instructions de virtualisation matérielle (Intel VMX, AMD SVM) et les structures (VMCS/VMCB, pagination imbriquée) dans le noyau de l'hôte. Cette émulation est complexe et contrôlée par l'invité, donc lorsque l'imbrication est activée elle constitue une large surface d'attaque supplémentaire du module KVM accessible depuis l'invité."
keywords:
  - virtualisation imbriquée
  - vmx
  - svm
  - vmcs
  - pagination imbriquée
---

# Virtualisation imbriquée

La virtualisation imbriquée permet à un invité d'être lui-même un hyperviseur exécutant des sous-invités. Pour la prendre en charge, KVM doit émuler la couche de virtualisation matérielle du CPU : les instructions Intel VMX ou AMD SVM (`VMLAUNCH`/`VMRESUME`, `VMREAD`/`VMWRITE`, `VMRUN`), les structures de contrôle (la VMCS sur Intel, la VMCB sur AMD), et la traduction par pagination imbriquée (EPT/NPT). Cette émulation est complexe et pilotée par des structures fournies par l'invité, donc activer l'imbrication expose une large surface supplémentaire dans le module noyau KVM, où une faille atterrit dans le noyau de l'hôte.

## La surface

```bash
# is nesting enabled on the host? (prerequisite for this surface)
cat /sys/module/kvm_intel/parameters/nested 2>/dev/null   # Y/1 => VMX nesting on
cat /sys/module/kvm_amd/parameters/nested 2>/dev/null      # AMD SVM nesting
```

```c
// with nesting on, an attacker guest acts as an L1 hypervisor and feeds KVM:
//  - a crafted VMCS/VMCB (control and guest-state fields) that KVM parses to set up
//    the nested entry; field handling and consistency-check bugs are reachable
//  - nested EPT/NPT paging structures KVM must shadow/translate; bad entries drive
//    the translation logic into out-of-bounds or type-confusion conditions
//  - VMREAD/VMWRITE to emulated VMCS fields and nested interrupt/event injection
```

Parce que KVM doit interpréter les structures de virtualisation fournies par L1 pour exécuter L2, l'invité contrôle des entrées complexes vers le code noyau, et les zones historiquement productives sont la gestion des champs VMCS et l'émulation de la pagination imbriquée.

## Notes d'exploitation

- Cette surface n'existe que lorsque l'imbrication est activée ; elle est désactivée par défaut sur certaines distributions, donc vérifiez d'abord le paramètre du module. Lorsqu'elle est activée, elle agrandit substantiellement la surface d'[Évasion par module noyau](kernel-module-escape.md).
- L'attaquant la pilote en exécutant un hyperviseur L1 minimal dans l'invité qui émet les opérations VMX/SVM avec des structures VMCS/VMCB et de pagination forgées, plutôt qu'un système d'exploitation imbriqué complet.
- Comme pour les autres bugs du module KVM, le succès est une exécution de code dans le noyau de l'hôte, contournant entièrement tout sandbox de moniteur en espace utilisateur.
- Spécifique à la version du noyau en cours d'exécution ; les chemins VMX et SVM sont du code distinct, donc le bug applicable dépend du fabricant du CPU de l'hôte.

## Références

- [KVM nested VMX documentation](https://docs.kernel.org/virt/kvm/x86/nested-vmx.html)
- [Intel SDM: VMX](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [Google Project Zero: nested virtualization research](https://googleprojectzero.blogspot.com/)
