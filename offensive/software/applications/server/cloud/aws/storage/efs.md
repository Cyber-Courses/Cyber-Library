---
title: "EFS: mounting exposed NFS file systems"
description: "Mounting exposed EFS file systems through permissive mount-target security groups or file-system policy."
keywords:
  - EFS
  - NFS
  - mount target
  - file system policy
  - share
---

# EFS

Elastic File System presents an NFS share reachable at a **mount target** inside a VPC. Its exposure is governed by the mount target's security group and, optionally, a file-system policy. When the security group allows NFS (TCP 2049) from a broad range, any instance you can place in that subnet, or any host that can route to it, mounts the file system and reads everything on it.

## Finding file systems and mount targets

```bash
aws efs describe-file-systems --query 'FileSystems[].[FileSystemId,Name]'
aws efs describe-mount-targets --file-system-id fs-xxxx \
  --query 'MountTargets[].[IpAddress,SubnetId,MountTargetId]'
aws efs describe-mount-target-security-groups --mount-target-id fsmt-xxxx
```

## Mounting from an instance in reach

```bash
# default EFS mount (no file-system policy enforcing IAM/TLS)
sudo mkdir /mnt/efs
sudo mount -t nfs4 -o nfsvers=4.1 <mount-target-ip>:/ /mnt/efs
ls -la /mnt/efs
```

## Exploitation notes

- EFS access without a restrictive file-system policy is decided purely by network reachability and POSIX permissions, so a foothold on any instance in the VPC usually suffices.
- Mounting as root yields root-squash-free access unless the share enforces it; read application data, home directories, and any secrets written to the share.
- Where a file-system policy requires IAM and TLS, use the `amazon-efs-utils` helper with an access point and a role you hold rather than a raw NFS mount.

## Tools

- **mount.nfs4** / **amazon-efs-utils**: the mount itself, raw or IAM-enforced.
- **AWS CLI** (`efs describe-*`): locate file systems, targets, and their security groups.

## References

- [HackTricks Cloud: EFS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-efs-enum.html)
- [AWS: EFS mount targets and security groups](https://docs.aws.amazon.com/efs/latest/ug/accessing-fs.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
