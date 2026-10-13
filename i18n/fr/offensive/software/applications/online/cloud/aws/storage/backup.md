---
title: "Backup : restaurer des points de récupération depuis les coffres AWS Backup"
order: 4
description: "Lire et restaurer des points de récupération depuis les coffres AWS Backup pour récupérer des données protégées."
keywords:
  - AWS Backup
  - recovery point
  - vault
  - restore
  - data recovery
---

# Backup

AWS Backup centralise les points de récupération d'EBS, RDS, DynamoDB, EFS et d'autres services dans des **coffres de sauvegarde** (backup vaults). Un principal disposant des permissions de lecture et de restauration des sauvegardes peut énumérer ces points de récupération et les restaurer dans des ressources qu'il contrôle, atteignant ainsi les données protégées sans toucher à la source en production. Lorsqu'une politique de coffre est trop permissive ou partagée entre comptes, les sauvegardes deviennent une porte dérobée vers tout ce qu'elles couvrent.

## Énumérer les coffres et les points de récupération

```bash
aws backup list-backup-vaults --query 'BackupVaultList[].BackupVaultName'
aws backup list-recovery-points-by-backup-vault --backup-vault-name <vault> \
  --query 'RecoveryPoints[].[RecoveryPointArn,ResourceType]'
aws backup get-backup-vault-access-policy --backup-vault-name <vault>
```

## Restaurer un point de récupération

```bash
# restore metadata tells you the required parameters for the resource type
aws backup get-recovery-point-restore-metadata \
  --backup-vault-name <vault> --recovery-point-arn <arn>

aws backup start-restore-job --recovery-point-arn <arn> \
  --iam-role-arn arn:aws:iam::<acct>:role/<restore-role> \
  --metadata file://restore-params.json
```

Le volume EBS, l'instance RDS ou la table DynamoDB restaurés sont à vous de lire.

## Notes d'exploitation

- La restauration contourne les contrôles appliqués à la ressource en production : une base de données que vous ne pouvez pas interroger directement devient lisible dès que son point de récupération est restauré dans une instance qui vous appartient.
- Une politique d'accès de coffre entre comptes permet de copier un point de récupération vers votre compte, ce qui défait l'isolation du compte source.
- `start-restore-job` a besoin d'un rôle qu'il peut transmettre ; un rôle de restauration faible est ici aussi un levier [PassRole](../identity/privilege-escalation/pass-role/index.md).

## Outils

- **AWS CLI** (`backup list-*`, `start-restore-job`) : énumérer et restaurer.
- **Pacu** (`backup__* ` modules) : énumération de coffres par session.

## Références

- [HackTricks Cloud : AWS Backup](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-backup-enum.html)
- [AWS : restaurer une sauvegarde](https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-a-backup.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
