---
order: 4
title: "Backdoor et VMCI : s'évader d'ESXi via les canaux de contrôle"
description: "La backdoor VMware est un port d'E/S que l'invité utilise pour émettre des commandes GuestRPC vers le processus vmx, sous-tendant des fonctionnalités de Tools comme les dossiers partagés et le glisser-déposer ; VMCI est un périphérique paravirtuel pour datagrammes et vSockets. Les deux analysent des requêtes contrôlées par l'invité dans vmx, donc des failles dans les gestionnaires de commandes RPC ou le traitement des datagrammes VMCI donnent l'exécution de code dans le processus hôte."
keywords:
  - port backdoor
  - guestrpc
  - rpci
  - vmci
  - hgfs
---

# Backdoor et VMCI

VMware expose deux chemins de contrôle de l'invité vers le processus `vmx` côté hôte. La backdoor héritée est un port d'E/S, `0x5658`, que l'invité pilote en chargeant la valeur magique `0x564D5868` (« VMXh ») dans EAX avec un sélecteur de commande dans ECX ; elle transporte le protocole GuestRPC/RPCI qui implémente les fonctionnalités de VMware Tools. VMCI, la Virtual Machine Communication Interface, est un périphérique PCI émulé fournissant des datagrammes et un transport vSockets. Les deux consomment des données fournies par l'invité dans `vmx`, de sorte qu'une faille d'analyse dans un gestionnaire de commande RPC (dossiers partagés, glisser-déposer, copier-coller) ou dans le traitement des datagrammes VMCI corrompt la mémoire de `vmx` et conduit à l'exécution de code sur l'hôte.

## La backdoor et GuestRPC

La backdoor est atteinte avec une convention de registres spécifique ; l'invité positionne la magie et la commande et exécute un `in`/`out` sur le port :

```c
// issue a backdoor command (simplified): magic in EAX, port 0x5658, cmd in CX
static inline void vmware_backdoor(uint32_t cmd, uint32_t arg,
                                   uint32_t *eax, uint32_t *ebx) {
    uint32_t a, b, c=cmd, d=0x5658;
    asm volatile("in %%dx, %%eax"
                 : "=a"(a), "=b"(b)
                 : "a"(0x564D5868), "b"(arg), "c"(c), "d"(d));
    *eax=a; *ebx=b;
}
// GuestRPC is layered on top: open a channel, then send RPCI command strings
// (BDOOR_CMD_MESSAGE), e.g. "info-get guestinfo.x" or the HGFS/DnD feature RPCs
```

Les gestionnaires à forte valeur sont les RPC de fonctionnalités : HGFS (dossiers partagés) analyse un protocole de requête dans `vmx`, et les gestionnaires de glisser-déposer et de copier-coller (DnD/CP) ont historiquement porté les bugs de sûreté mémoire utilisés dans les évasions guest-to-host. Un invité émet des messages RPCI forgés pour piloter un gestionnaire vulnérable.

## VMCI

VMCI est un périphérique PCI que le pilote invité programme pour envoyer des datagrammes et établir des queue pairs et des connexions vSocket. Le côté hôte traite les en-têtes de datagrammes et la mise en place des queue pairs depuis une mémoire contrôlée par l'invité :

```bash
# the guest VMCI device and driver
lspci -nn | grep -i vmci
dmesg | grep -i vmw_vmci
# datagrams are sent via the device's hypercall/MMIO interface the driver exposes
```

Des failles dans la gestion des datagrammes ou des queue pairs VMCI, et dans la couche vSockets, corrompent `vmx` (ou dans certains chemins le module noyau) depuis l'invité.

## Notes d'exploitation

- Le port backdoor brut est accessible sans VMware Tools, donc la surface GuestRPC de base est disponible pour tout invité pouvant exécuter des E/S privilégiées ; les RPC de fonctionnalités plus riches supposent Tools mais un invité peut parler le protocole directement.
- Les gestionnaires HGFS et DnD/CP sont les cibles classiques car ils analysent des structures invité complexes et imbriquées dans `vmx` ; un bug y donne un hors-limites ou un use-after-free dans le tas du processus hôte.
- Une évasion complète enchaîne une fuite d'information (pour défaire l'ASLR dans `vmx`) avec la primitive de corruption mémoire, puis pivote de l'exécution de code dans `vmx` vers l'hôte ; elles sont spécifiques à la version, donc identifiez d'abord le build ESXi.
- Les produits de bureau exposent la même surface GuestRPC/HGFS, raison pour laquelle les bugs de dossiers partagés se retrouvent dans Workstation, Fusion et ESXi ; voir [Workstation et Fusion](../../workstation-and-fusion/index.md).

## Références

- [Zero Day Initiative: VMware GuestRPC and vmx research](https://www.zerodayinitiative.com/blog)
- [VMware backdoor protocol (community documentation)](https://sites.google.com/site/chitchatvmback/backdoor)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
