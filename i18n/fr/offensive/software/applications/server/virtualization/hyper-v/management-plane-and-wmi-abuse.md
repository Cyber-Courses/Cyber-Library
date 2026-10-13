---
order: 2
title: "Plan de gestion et abus de WMI : contrôler Hyper-V via ses API"
description: "Hyper-V se gère via WMI (l'espace de noms root/virtualization), des cmdlets PowerShell et des services distants. Un attaquant disposant de droits sur cette surface de gestion contrôle chaque VM, crée ou modifie des VM pour monter des disques accessibles depuis l'hôte ou exécuter des payloads, utilise PowerShell Direct vers les invités, et pose des abonnements d'événements WMI comme persistance sur l'hôte."
keywords:
  - hyper-v wmi
  - root/virtualization
  - powershell direct
  - abonnement d'événements wmi
  - plan de gestion
---

# Plan de gestion et abus de WMI

La surface de contrôle de Hyper-V est programmatique : l'espace de noms WMI `root/virtualization/v2`, le module PowerShell Hyper-V, et les services de gestion distante (WinRM, RPC/DCOM) qui les exposent. Un attaquant capable d'atteindre cette surface avec des droits suffisants contrôle le parc de virtualisation sans évasion d'invité : énumérer et manipuler des VM, créer ou reconfigurer une VM pour monter un disque ou exécuter un payload, piloter des invités avec PowerShell Direct, et utiliser l'événementiel de WMI comme persistance durable sur l'hôte.

## Piloter Hyper-V via WMI et PowerShell

```powershell
# enumerate and control VMs via the Hyper-V module (wraps the WMI namespace)
Get-VM; Get-VMHardDiskDrive -VMName *
# or directly through WMI
Get-CimInstance -Namespace root\virtualization\v2 -ClassName Msvm_ComputerSystem
# create/modify a VM to attach a target VHD or boot attacker media
New-VM -Name x -VHDPath '<target>.vhdx'          # attach another VM's disk
# run inside a guest from the host, no guest network needed
Invoke-Command -VMName <guest> -Credential <c> -ScriptBlock { ... }
```

## WMI comme persistance

```powershell
# a permanent WMI event subscription runs a payload on a trigger, as SYSTEM, and
# survives reboots - a classic host persistence mechanism on the Hyper-V host
# (__EventFilter + CommandLineEventConsumer + FilterToConsumerBinding)
```

## Notes d'exploitation

- Le plan de gestion est une alternative à une évasion d'invité : avec l'administration de l'hôte ou des droits Hyper-V délégués, vous contrôlez chaque VM et pouvez attacher le disque d'une autre VM à une VM que vous contrôlez pour le lire, ou démarrer une VM depuis un média de l'attaquant.
- PowerShell Direct est le pivot discret de l'hôte vers l'invité : il s'exécute dans l'invité via le canal adossé à VMBus sans exposition réseau de l'invité, nécessitant seulement des identifiants invité et un accès hôte.
- Les abonnements d'événements WMI permanents constituent une persistance durable de niveau SYSTEM sur l'hôte, et se fondent dans l'outillage administratif.
- La gestion déléguée (groupe Hyper-V Administrators, délégation contrainte) peut accorder le contrôle des VM sans administration complète de l'hôte ; énumérez qui en dispose comme aide au ciblage.

## Références

- [Microsoft: Hyper-V WMI provider (root/virtualization)](https://learn.microsoft.com/en-us/windows/win32/hyperv_v2/windows-virtualization-portal)
- [Microsoft: Hyper-V PowerShell module](https://learn.microsoft.com/en-us/powershell/module/hyper-v/)
- [MITRE ATT&CK: WMI event subscription](https://attack.mitre.org/techniques/T1546/003/)
