---
order: 1
title: "Hyper-V : attaquer l'hyperviseur Microsoft"
description: "Hyper-V est un hyperviseur de type 1 où les invités communiquent avec la partition racine via VMBus. La surface d'évasion guest-to-host est constituée des périphériques synthétiques et des canaux VMBus servis par les VSP dans la partition racine et le processus worker, ainsi que du plan de gestion via WMI et PowerShell, et du vol des checkpoints et des disques virtuels VHD."
keywords:
  - hyper-v
  - vmbus
  - partition racine
  - vmwp
  - périphériques synthétiques
---

# Hyper-V

Hyper-V est l'hyperviseur de type 1 de Microsoft : un hyperviseur fin s'exécute sous une partition racine privilégiée, et les partitions invitées atteignent les services de l'hôte via VMBus, un transport de canal par ring-buffer. Chaque VM a un processus worker (`vmwp.exe`) dans la partition racine, et les périphériques synthétiques qu'utilise un invité sont servis par des Virtualization Service Providers (VSP) là. La surface d'évasion guest-to-host est cette gestion de VMBus et des périphériques synthétiques dans la partition racine ; à ses côtés se trouvent le plan de gestion (cmdlets WMI et PowerShell) et les artefacts sur disque, checkpoints et disques virtuels VHD/VHDX, qui contiennent les données des invités.

```powershell
# from a Windows guest: the synthetic devices and VMBus
Get-PnpDevice | Where-Object { $_.InstanceId -like 'VMBUS*' }
# from the host: VMs, their worker processes, and management
Get-VM; Get-Process vmwp
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape/index.md)** : breakout VMBus et périphériques synthétiques vers la partition racine.
- **[Vol de checkpoints et de VHD](checkpoint-and-vhd-theft.md)** : voler les disques virtuels et l'état sauvegardé.
- **[Accès à l'hôte et shell](host-access-and-shell.md)** : exécution sur l'hôte Hyper-V.
- **[Plan de gestion et abus de WMI](management-plane-and-wmi-abuse.md)** : la surface de contrôle WMI et PowerShell.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs récurrents VMBus et VSP.

## Références

- [Microsoft: Hyper-V architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V bug bounty and research](https://www.microsoft.com/en-us/msrc)
- [Microsoft security update guide](https://msrc.microsoft.com/update-guide)
