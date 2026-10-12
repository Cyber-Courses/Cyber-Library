---
title: "Partages de fichiers : atteindre Azure Files via SMB"
order: 4
description: "Atteindre les partages SMB d'Azure Files via des clés de stockage ou un accès basé sur l'identité pour récupérer les données stockées."
keywords:
  - Azure Files
  - SMB
  - file share
  - storage key
  - data
---

# Partages de fichiers

Azure Files expose des partages SMB (et NFS) adossés à un compte de stockage, montés par des serveurs et des postes de travail pour les répertoires personnels, les données applicatives et les sauvegardes. L'accès se fait par la **clé de compte de stockage** (qui s'authentifie directement auprès du partage) ou par une authentification **basée sur l'identité** (Entra Kerberos ou AD DS on-prem) mappée au RBAC. Une clé de compte récupérée est la voie la plus rapide : elle monte n'importe quel partage du compte.

## Lister et lire les partages

```bash
# enumerate shares and files with the account key
az storage share list --account-name acme --account-key <KEY>
az storage file download --account-name acme --account-key <KEY> \
  --share-name profiles --path 'user/secrets.config' --dest ./secrets.config
```

## Monter via SMB

```bash
# mount the share directly with the account key as the password
mount -t cifs //acme.file.core.windows.net/profiles /mnt/share \
  -o vers=3.0,username=acme,password=<KEY>,dir_mode=0777,file_mode=0777
```

## Notes d'exploitation

- La clé de compte sert aussi de mot de passe SMB : `listkeys` sur le compte de stockage donne donc un accès complet au partage sans aucune permission spécifique aux partages de fichiers.
- Azure Files est accessible depuis Internet sur le port 445 lorsque le compte n'a aucune restriction réseau, de sorte qu'une clé plus un compte ouvert constituent un accès aux données à distance sans prise de pied dans le VNet.
- Recherchez les partages montés référencés dans le `fstab`, les scripts et les tâches planifiées des VM pour identifier ceux qui détiennent les données précieuses.

## Outils

- **az CLI** (`storage share`, `storage file`) : lister et extraire des fichiers.
- **cifs-utils** (`mount -t cifs`) : monter le partage avec la clé.
- **MicroBurst** : récupère les clés de compte qui déverrouillent les partages.

## Références

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [Microsoft: mount Azure Files on Linux](https://learn.microsoft.com/azure/storage/files/storage-how-to-use-files-linux)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
