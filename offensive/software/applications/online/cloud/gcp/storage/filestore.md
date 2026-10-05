---
title: "Filestore: reaching NFS shares inside the VPC"
description: "Mounting or reading Filestore NFS shares exposed within the VPC to reach other workloads' data."
keywords:
  - Filestore
  - NFS
  - file share
  - VPC
  - mount
---

# Filestore

Filestore is managed NFS. Its access control is network reachability plus standard NFS semantics, not Cloud IAM, so any workload that sits in the same VPC (or a peered one) and reaches the instance IP can mount the share and read whatever it holds. A foothold on any VM in the network is often enough to reach another team's Filestore data.

## Finding and mounting a share

```bash
# enumerate Filestore instances and their export IP and share name
gcloud filestore instances list
gcloud filestore instances describe <instance> --zone <zone> \
  --format='value(networks[0].ipAddresses[0],fileShares[0].name)'

# from a VM in the VPC, mount the export
sudo apt-get install -y nfs-common
sudo mkdir /mnt/fs && sudo mount <filestore-ip>:/<share> /mnt/fs
ls -la /mnt/fs
```

## Exploitation notes

- No credential is checked at mount time beyond network reach and NFS uid/gid, so matching the owning uid (or mounting as root) reads restricted files.
- Shares frequently hold home directories, backups, and application state with embedded secrets; triage the same way as a looted disk.
- Reachability follows VPC peering and firewall rules, so a share can be exposed to more of the network than its owners expect.

## Tools

- **gcloud filestore**: instance and export enumeration.
- **nfs-common / mount**: mounting the share from a VPC host.

## References

- [HackTricks Cloud: GCP Filestore](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: mounting Filestore file shares](https://cloud.google.com/filestore/docs/mounting-fileshares)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
