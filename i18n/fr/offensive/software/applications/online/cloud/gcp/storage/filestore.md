---
title: "Filestore : atteindre les partages NFS à l'intérieur du VPC"
description: "Monter ou lire des partages NFS Filestore exposés au sein du VPC pour accéder aux données d'autres workloads."
keywords:
  - Filestore
  - NFS
  - partage de fichiers
  - VPC
  - montage
---

# Filestore

Filestore est un service NFS managé. Son contrôle d'accès repose sur l'accessibilité réseau et la sémantique NFS standard, et non sur Cloud IAM, donc tout workload situé dans le même VPC (ou un VPC appairé) et pouvant atteindre l'IP de l'instance peut monter le partage et lire tout ce qu'il contient. Un point d'ancrage sur n'importe quelle VM du réseau suffit souvent à atteindre les données Filestore d'une autre équipe.

## Trouver et monter un partage

```bash
# enumerate Filestore instances and their export IP and share name
gcloud filestore instances list
gcloud filestore instances describe <instance> --zone <zone> \
  --format='value(networks[0].ipAddresses[0],fileShares[0].name)'

# from a VM in the VPC, mount the export
sudo apt-get install -y nfs-common
sudo mkdir /mnt/fs && sudo mount <filestore-ip>:/<share> /mnt/fs
ls -la /mnt/fs
```

## Notes d'exploitation

- Aucune identification n'est vérifiée au moment du montage au-delà de l'accessibilité réseau et de l'uid/gid NFS, donc faire correspondre l'uid propriétaire (ou monter en tant que root) permet de lire les fichiers restreints.
- Les partages contiennent fréquemment des répertoires personnels, des sauvegardes et des états d'application avec des secrets intégrés ; triez-les de la même manière qu'un disque pillé.
- L'accessibilité suit l'appairage VPC et les règles de pare-feu, donc un partage peut être exposé à une plus grande partie du réseau que ce que ses propriétaires pensent.

## Outils

- **gcloud filestore** : énumération des instances et des exports.
- **nfs-common / mount** : montage du partage depuis un hôte du VPC.

## Références

- [HackTricks Cloud: GCP Filestore](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: mounting Filestore file shares](https://cloud.google.com/filestore/docs/mounting-fileshares)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
