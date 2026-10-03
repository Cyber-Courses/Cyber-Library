---
title: "FSx: reaching Windows, Lustre, and NetApp file systems"
description: "Accessing FSx for Windows, Lustre, or NetApp file systems exposed through share permissions and security groups."
keywords:
  - FSx
  - file system
  - SMB
  - Lustre
  - share
---

# FSx

FSx provides managed file systems in several flavours (Windows File Server over SMB, Lustre, OpenZFS, and NetApp ONTAP), each reachable over its native protocol inside a VPC. As with EFS, exposure comes down to the security group on the file system's network interface and the share's own permissions. A reachable SMB or NFS endpoint with weak share permissions hands over the data.

## Finding file systems

```bash
aws fsx describe-file-systems \
  --query 'FileSystems[].[FileSystemId,FileSystemType,DNSName]'
aws fsx describe-file-systems --query 'FileSystems[].NetworkInterfaceIds'
```

## Mounting the share

```bash
# FSx for Windows (SMB) from a reachable host
mount -t cifs //<dns-name>/share /mnt/fsx -o user=<user>,dom=<domain>

# FSx for Lustre
sudo mount -t lustre <dns-name>@tcp:/<mountname> /mnt/fsx
```

## Exploitation notes

- FSx for Windows joins an Active Directory domain; a domain credential from the directory side often unlocks the share, linking the cloud and on-premises attack paths.
- Lustre and OpenZFS mounts are gated mostly by network reachability, so an instance in the right subnet is usually enough.
- Backups of FSx file systems also land in [AWS Backup](backup.md) and are restorable from there.

## Tools

- **mount.cifs** / **mount.lustre**: protocol-native mounts.
- **AWS CLI** (`fsx describe-file-systems`): locate file systems and their interfaces.

## References

- [HackTricks Cloud: FSx](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-fsx-enum.html)
- [AWS: accessing FSx file systems](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/using-file-shares.html)
