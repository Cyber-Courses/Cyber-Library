---
order: 1
title: "Évasion guest-to-host : sortir d'une machine virtuelle Hyper-V"
description: "Un invité Hyper-V s'évade en corrompant le code de la partition racine qui traite ses requêtes : le transport de canal VMBus et les périphériques synthétiques (stockage, réseau, vidéo, HID) adossés aux Virtualization Service Providers, ainsi que les périphériques émulés et le commutateur virtuel dans le processus worker. Une faille de sûreté mémoire à cet endroit s'exécute dans la partition racine, qui contrôle chaque VM."
keywords:
  - évasion hyper-v
  - vmbus
  - vsp
  - périphériques synthétiques
  - partition racine
---

# Évasion guest-to-host

Un invité Hyper-V est isolé par l'hyperviseur, il ne peut donc pas toucher l'hôte directement ; il atteint les fonctionnalités de l'hôte via VMBus, par lequel les périphériques synthétiques dialoguent avec les Virtualization Service Providers (VSP) dans la partition racine, et via le processus worker (`vmwp.exe`) qui gère les périphériques émulés et le commutateur virtuel. S'évader signifie faire manipuler incorrectement par l'un de ces composants de la partition racine des données contrôlées par l'invité : un paquet VMBus, une requête de périphérique synthétique ou un accès à un périphérique émulé. L'exécution de code atterrit dans la partition racine ou le processus worker, ce qui équivaut au contrôle de l'hôte car la partition racine administre tous les invités.

```powershell
# the VMBus devices and channels visible from the guest
Get-PnpDevice | Where-Object InstanceId -like 'VMBUS*'
# synthetic storage/net/video/HID are the high-level targets; the transport is VMBus
```

## Sous-thèmes

- **[VMBus](vmbus.md)** : le transport de canal par ring-buffer entre l'invité et la partition racine.
- **[Périphériques synthétiques](synthetic-devices.md)** : les périphériques de stockage, réseau, vidéo et HID adossés aux VSP.
- **[Commutateur virtuel](virtual-switch.md)** : le datapath réseau dans la partition racine.

## Références

- [Microsoft: Hyper-V architecture and VMBus](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V security research](https://www.microsoft.com/en-us/msrc)
