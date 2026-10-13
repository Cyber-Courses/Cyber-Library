---
order: 3
title: "Vol de checkpoints et de VHD : récupérer les disques virtuels et l'état sauvegardé de Hyper-V"
description: "Hyper-V stocke les disques des VM sous forme de fichiers VHD/VHDX et les checkpoints sous forme de disques différentiels accompagnés de fichiers d'état sauvegardé et de mémoire. Avec un accès à l'hôte ou au partage, un attaquant les copie et lit les systèmes de fichiers des invités hors ligne, extrayant identifiants et données sans entrer dans la VM, et lit la mémoire d'état sauvegardé pour des secrets capturés au moment du checkpoint."
keywords:
  - vhd
  - vhdx
  - checkpoint
  - état sauvegardé
  - accès hors ligne
---

# Vol de checkpoints et de VHD

Hyper-V conserve les disques de chaque VM sous forme de fichiers VHD ou VHDX et ses checkpoints sous forme de disques différentiels (`.avhdx`) aux côtés des fichiers de configuration et d'état sauvegardé. L'accès à ces fichiers, sur l'hôte, un partage de bibliothèque SMB ou une sauvegarde, permet à un attaquant de lire les systèmes de fichiers des invités hors ligne et d'en extraire tout le contenu sans toucher à la VM en cours d'exécution : aucune connexion, aucune défense dans l'invité. Les fichiers d'état sauvegardé et de mémoire issus d'un checkpoint capturent en outre la mémoire vive, de sorte que les secrets présents lors de la prise du checkpoint sont récupérables.

## Localiser et monter les disques

```powershell
# find VM disks and checkpoints
Get-VM | Get-VMHardDiskDrive | Select VMName, Path
Get-VMSnapshot -VMName <vm>                        # checkpoints -> .avhdx differencing disks
# default locations
dir 'C:\ProgramData\Microsoft\Windows\Hyper-V\'
dir 'C:\Users\Public\Documents\Hyper-V\Virtual hard disks\'
# mount a VHDX read-only to read the guest filesystem offline
Mount-VHD -Path '<vm>.vhdx' -ReadOnly
# then read the mounted volume: SAM/SYSTEM hives, files, keys
```

```bash
# offline on another host with libguestfs
guestmount -a disk.vhdx -i --ro /mnt/guest
reg="/mnt/guest/Windows/System32/config"; ls "$reg"/SAM "$reg"/SYSTEM
```

## État sauvegardé et mémoire

```powershell
# saved-state (.vsv) and memory (.bin / .vmrs) files hold live memory at save time
# carve them for in-memory secrets (keys, tokens, cached credentials)
dir '<vm path>\*.VMRS','<vm path>\*.bin'
```

## Notes d'exploitation

- Monter un VHDX en lecture seule (ou `guestmount` hors ligne) contourne tous les contrôles internes à l'invité ; priorisez les ruches Windows SAM/SYSTEM pour l'extraction de hash hors ligne et toute clé stockée.
- Les disques de checkpoint différentiels (`.avhdx`) doivent être chaînés au parent pour une vue complète du système de fichiers ; montez toute la chaîne, ou fusionnez, pour lire l'état courant.
- Les fichiers d'état sauvegardé et de mémoire équivalent à une image mémoire au moment du checkpoint et contiennent des secrets qui étaient déchiffrés en RAM ; extrayez-les avec un outil de forensique mémoire.
- L'accès aux fichiers est la condition préalable complète : un partage de stockage Hyper-V ou une sauvegarde lisible équivaut au vol des données invitées de chaque VM qui s'y trouve, sans aucune exécution sur l'hôte.

## Outils

- [libguestfs / guestmount](https://libguestfs.org/)

## Références

- [Microsoft: VHDX format and checkpoints](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/)
- [Microsoft: manage Hyper-V checkpoints](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/checkpoints)
