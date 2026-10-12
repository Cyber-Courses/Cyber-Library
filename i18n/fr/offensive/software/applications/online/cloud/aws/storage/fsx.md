---
title: "FSx : atteindre les systèmes de fichiers Windows, Lustre et NetApp"
order: 5
description: "Accéder aux systèmes de fichiers FSx for Windows, Lustre ou NetApp exposés via les permissions de partage et les groupes de sécurité."
keywords:
  - FSx
  - file system
  - SMB
  - Lustre
  - share
---

# FSx

FSx fournit des systèmes de fichiers managés dans plusieurs variantes (Windows File Server via SMB, Lustre, OpenZFS et NetApp ONTAP), chacune accessible via son protocole natif à l'intérieur d'un VPC. Comme pour EFS, l'exposition se résume au groupe de sécurité de l'interface réseau du système de fichiers et aux permissions propres du partage. Un point de terminaison SMB ou NFS accessible avec des permissions de partage faibles livre les données.

## Trouver les systèmes de fichiers

```bash
aws fsx describe-file-systems \
  --query 'FileSystems[].[FileSystemId,FileSystemType,DNSName]'
aws fsx describe-file-systems --query 'FileSystems[].NetworkInterfaceIds'
```

## Monter le partage

```bash
# FSx for Windows (SMB) from a reachable host
mount -t cifs //<dns-name>/share /mnt/fsx -o user=<user>,dom=<domain>

# FSx for Lustre
sudo mount -t lustre <dns-name>@tcp:/<mountname> /mnt/fsx
```

## Notes d'exploitation

- FSx for Windows rejoint un domaine Active Directory ; un identifiant de domaine venu du côté annuaire débloque souvent le partage, reliant les chemins d'attaque cloud et sur site.
- Les montages Lustre et OpenZFS sont principalement contrôlés par l'accessibilité réseau, une instance dans le bon sous-réseau suffit donc généralement.
- Les sauvegardes des systèmes de fichiers FSx atterrissent également dans [AWS Backup](backup.md) et y sont restaurables.

## Outils

- **mount.cifs** / **mount.lustre** : montages natifs au protocole.
- **AWS CLI** (`fsx describe-file-systems`) : localiser les systèmes de fichiers et leurs interfaces.

## Références

- [HackTricks Cloud : FSx](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-fsx-enum.html)
- [AWS : accéder aux systèmes de fichiers FSx](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/using-file-shares.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
