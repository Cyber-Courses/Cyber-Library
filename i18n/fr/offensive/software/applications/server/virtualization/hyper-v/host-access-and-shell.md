---
order: 4
title: "Accès à l'hôte et shell : exécution sur l'hôte Hyper-V"
description: "Un hôte Hyper-V est une machine Windows Server (ou client), donc l'accès à l'hôte est un accès Windows : identifiants administratifs, interfaces de gestion, ou une évasion d'invité atterrissant dans la partition racine. Avec cela, un attaquant contrôle chaque VM via la gestion Hyper-V, lit les disques des VM, injecte dans les invités et persiste sur l'hôte comme sur n'importe quel système Windows."
keywords:
  - hôte hyper-v
  - partition racine
  - windows server
  - hyper-v manager
  - persistance
---

# Accès à l'hôte et shell

Un hôte Hyper-V est une machine Windows exécutant le rôle, donc atteindre l'hôte revient à atteindre Windows : via des identifiants administratifs et les interfaces distantes habituelles (RDP, WinRM, SMB, WMI), via la pile de gestion Hyper-V, ou via une évasion d'invité qui atterrit dans la partition racine. La partition racine est une partition Windows privilégiée qui administre chaque invité, donc l'exécution de code sur l'hôte signifie le contrôle complet de toutes les VM, de leurs disques et de la configuration de virtualisation, avec des options de persistance identiques à celles de n'importe quel serveur Windows.

## Atteindre et utiliser l'hôte

```powershell
# remote execution with host admin credentials
Enter-PSSession -ComputerName <host> -Credential <admin>
winrs -r:<host> cmd                              # or WinRM / PsExec / WMI
# once on the host, Hyper-V management controls every VM
Get-VM; Get-VMHardDiskDrive -VMName *
# run commands inside a guest from the host (with guest creds or via PowerShell Direct)
Invoke-Command -VMName <guest> -ScriptBlock { whoami } -Credential <guestcred>
```

PowerShell Direct (`-VMName`) exécute des commandes dans un invité depuis l'hôte sans accès réseau à l'invité, en utilisant la position privilégiée de l'hôte, ce qui constitue un pivot propre de l'hôte vers l'invité.

## Persistance

```powershell
# the host is Windows: standard persistence applies
#  - a service, scheduled task, or Run key
#  - a WMI event subscription (see management-plane-and-wmi-abuse)
# plus virtualization-specific leverage: modify a VM's config or inject via its disk
```

## Notes d'exploitation

- L'accès à l'hôte englobe tout le reste : depuis la partition racine vous lisez et modifiez les disques des VM ([Vol de checkpoints et de VHD](checkpoint-and-vhd-theft.md)) et exécutez à l'intérieur des invités via PowerShell Direct sans avoir besoin d'une portée réseau sur l'invité.
- Une évasion guest-to-host fait atterrir du code dans la partition racine ou le processus worker ; de là, l'élévation de privilèges locale vers l'administration complète de l'hôte suit le playbook Windows normal si l'on n'est pas déjà SYSTEM.
- La persistance est une persistance Windows standard ; l'ajout spécifique à la virtualisation est l'altération des configurations et disques des VM pour qu'une VM cible exécute du contenu de l'attaquant au prochain démarrage.
- Le plan de gestion (WMI/PowerShell) est à la fois une entrée distante et une surface de persistance ; voir [Plan de gestion et abus de WMI](management-plane-and-wmi-abuse.md).

## Références

- [Microsoft: Hyper-V management](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/)
- [Microsoft: PowerShell Direct](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/powershell-direct)
