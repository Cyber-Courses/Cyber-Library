---
title: "EFS : monter des systèmes de fichiers NFS exposés"
order: 3
description: "Monter des systèmes de fichiers EFS exposés via des groupes de sécurité de cible de montage ou une politique de système de fichiers trop permissifs."
keywords:
  - EFS
  - NFS
  - mount target
  - file system policy
  - share
---

# EFS

Elastic File System présente un partage NFS accessible au niveau d'une **cible de montage** (mount target) à l'intérieur d'un VPC. Son exposition est régie par le groupe de sécurité de la cible de montage et, éventuellement, par une politique de système de fichiers. Lorsque le groupe de sécurité autorise NFS (TCP 2049) depuis une plage trop large, toute instance que vous pouvez placer dans ce sous-réseau, ou tout hôte capable d'y accéder par routage, monte le système de fichiers et lit tout ce qu'il contient.

## Trouver les systèmes de fichiers et les cibles de montage

```bash
aws efs describe-file-systems --query 'FileSystems[].[FileSystemId,Name]'
aws efs describe-mount-targets --file-system-id fs-xxxx \
  --query 'MountTargets[].[IpAddress,SubnetId,MountTargetId]'
aws efs describe-mount-target-security-groups --mount-target-id fsmt-xxxx
```

## Monter depuis une instance à portée

```bash
# default EFS mount (no file-system policy enforcing IAM/TLS)
sudo mkdir /mnt/efs
sudo mount -t nfs4 -o nfsvers=4.1 <mount-target-ip>:/ /mnt/efs
ls -la /mnt/efs
```

## Notes d'exploitation

- L'accès à EFS, en l'absence d'une politique de système de fichiers restrictive, est décidé uniquement par l'accessibilité réseau et les permissions POSIX ; un point d'appui sur n'importe quelle instance du VPC suffit donc généralement.
- Monter en tant que root donne un accès sans root-squash à moins que le partage ne l'impose ; lisez les données applicatives, les répertoires personnels et tout secret écrit sur le partage.
- Lorsqu'une politique de système de fichiers exige IAM et TLS, utilisez l'utilitaire `amazon-efs-utils` avec un point d'accès et un rôle que vous détenez plutôt qu'un montage NFS brut.

## Outils

- **mount.nfs4** / **amazon-efs-utils** : le montage lui-même, brut ou renforcé par IAM.
- **AWS CLI** (`efs describe-*`) : localiser les systèmes de fichiers, les cibles et leurs groupes de sécurité.

## Références

- [HackTricks Cloud : EFS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-efs-enum.html)
- [AWS : cibles de montage EFS et groupes de sécurité](https://docs.aws.amazon.com/efs/latest/ug/accessing-fs.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
