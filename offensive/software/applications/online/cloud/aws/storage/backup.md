---
title: "Backup: restoring recovery points from AWS Backup vaults"
order: 4
description: "Reading and restoring recovery points from AWS Backup vaults to recover protected data."
keywords:
  - AWS Backup
  - recovery point
  - vault
  - restore
  - data recovery
---

# Backup

AWS Backup centralises recovery points for EBS, RDS, DynamoDB, EFS, and more into **backup vaults**. A principal with backup read and restore permissions can enumerate those recovery points and restore them into resources it controls, reaching the protected data without touching the live source. Where a vault policy is over-broad or shared cross-account, the backups become an alternate door to everything they cover.

## Enumerating vaults and recovery points

```bash
aws backup list-backup-vaults --query 'BackupVaultList[].BackupVaultName'
aws backup list-recovery-points-by-backup-vault --backup-vault-name <vault> \
  --query 'RecoveryPoints[].[RecoveryPointArn,ResourceType]'
aws backup get-backup-vault-access-policy --backup-vault-name <vault>
```

## Restoring a recovery point

```bash
# restore metadata tells you the required parameters for the resource type
aws backup get-recovery-point-restore-metadata \
  --backup-vault-name <vault> --recovery-point-arn <arn>

aws backup start-restore-job --recovery-point-arn <arn> \
  --iam-role-arn arn:aws:iam::<acct>:role/<restore-role> \
  --metadata file://restore-params.json
```

The restored EBS volume, RDS instance, or DynamoDB table is yours to read.

## Exploitation notes

- Restoring sidesteps controls on the live resource: a database you cannot query directly is readable once its recovery point is restored into an instance you own.
- A cross-account vault access policy lets a recovery point be copied to your account, defeating source-account isolation.
- `start-restore-job` needs a role it can pass; a weak restore role here is also a [PassRole](../identity/privilege-escalation/pass-role/index.md) lever.

## Tools

- **AWS CLI** (`backup list-*`, `start-restore-job`): enumerate and restore.
- **Pacu** (`backup__* ` modules): session-based vault enumeration.

## References

- [HackTricks Cloud: AWS Backup](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-backup-enum.html)
- [AWS: restoring a backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-a-backup.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
