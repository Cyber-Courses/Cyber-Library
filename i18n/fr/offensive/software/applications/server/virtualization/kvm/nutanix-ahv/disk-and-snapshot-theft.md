---
order: 4
title: "Vol de disques et de snapshots : récupérer les vdisks depuis la fabrique de stockage Nutanix"
description: "Nutanix stocke les disques de chaque VM sous forme de vdisks dans l'Acropolis Distributed Storage Fabric, accessibles via les Controller VM. Avec un accès CVM ou Prism, un attaquant clone ou snapshote un vdisk cible, l'exporte en image, et lit le système de fichiers de l'invité hors ligne, récupérant les données de n'importe quelle VM sans y entrer, et lit les snapshots pour un état à un instant donné."
keywords:
  - vdisk
  - distributed storage fabric
  - snapshot
  - export d'image
  - nutanix
---

# Vol de disques et de snapshots

Nutanix conserve les disques de chaque VM sous forme de vdisks dans l'Acropolis Distributed Storage Fabric, servis par les Controller VM plutôt que comme de simples fichiers sur un seul hôte. Le chemin de vol passe donc par le CVM ou Prism plutôt que par la copie d'un fichier depuis un datastore : un attaquant disposant de cet accès clone ou snapshote le vdisk cible, l'exporte en image (ou attache le clone à une VM qu'il contrôle), et lit le système de fichiers de l'invité hors ligne. Les snapshots donnent des copies à un instant donné, et la technique de clone-et-attache lit le disque de n'importe quelle VM sans entrer dans cette VM.

## Cloner, exporter et lire

```bash
# via the CVM CLI: snapshot/clone a target VM's disk, then export or attach it
acli vm.list                                   # find the target VM
acli snapshot.create <vm>                       # point-in-time snapshot
acli vm.clone <attacker-vm> clone_from_vm=<vm>  # clone whose disks you can mount
# or attach the cloned vdisk to a VM you control and read it from inside
# via Prism v3: clone the vdisk / create an image from it, then download the image
```

```bash
# once you have the image/clone attached to a controlled VM, read it offline
guestmount -a /dev/<attached-disk> -i --ro /mnt/guest
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
```

## Notes d'exploitation

- L'accès à la fabrique passe par le CVM/Prism, pas par une copie de fichier : les techniques pratiques sont le snapshot, le clone ou l'export d'image du vdisk cible, puis l'attache du résultat à une VM contrôlée ou le téléchargement de l'image exportée.
- Attacher un clone à une VM contrôlée par l'attaquant et le monter hors ligne contourne tous les contrôles internes à l'invité, exactement comme le vol de VMDK/qcow2 basé sur fichier ailleurs ; priorisez les magasins d'identifiants.
- Les snapshots capturent un état à un instant donné ; une VM avec des snapshots application-consistent ou mémoire peut exposer davantage. L'export d'image produit une copie portable à exfiltrer.
- Cela nécessite un accès CVM ou Prism, donc cela s'enchaîne depuis [Accès à l'hôte et shell](host-access-and-shell.md) et le [Plan de gestion](management-plane.md).

## Outils

- [libguestfs / guestmount](https://libguestfs.org/)

## Références

- [Nutanix: snapshots and clones](https://portal.nutanix.com/)
- [Acropolis Distributed Storage Fabric](https://www.nutanix.com/products/ahv)
