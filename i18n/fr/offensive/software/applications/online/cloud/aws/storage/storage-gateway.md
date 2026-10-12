---
title: "Storage Gateway : abuser du pont entre l'on-premises et S3/EBS"
order: 6
description: "Abus des partages de fichiers et des volumes en cache de Storage Gateway qui relient l'accès on-premises à S3 et EBS."
keywords:
  - Storage Gateway
  - file share
  - volume
  - S3
  - hybrid
---

# Storage Gateway

Storage Gateway relie les hôtes on-premises au stockage AWS : les file gateways exposent les buckets S3 comme des partages NFS ou SMB, et les volume gateways présentent des volumes iSCSI adossés à EBS. La gateway détient un accès permanent au S3 et à l'EBS sous-jacents, donc atteindre le partage d'une gateway, ou la configuration du plan de contrôle de la gateway elle-même, constitue une voie indirecte vers le stockage cloud qui se trouve derrière.

## Énumérer les gateways et les partages

```bash
aws storagegateway list-gateways --query 'Gateways[].[GatewayId,GatewayType]'
aws storagegateway list-file-shares --query 'FileShareInfoList[].FileShareARN'
aws storagegateway describe-nfs-file-shares --file-share-arn-list <arn> \
  --query 'NFSFileShareInfoList[].[LocationARN,ClientList,Squash]'
aws storagegateway list-volumes
```

Le `LocationARN` d'un file share nomme le bucket S3 qu'il expose ; la `ClientList` et les paramètres de squash vous indiquent qui peut le monter.

## Atteindre les données

```bash
# mount an NFS file gateway share exposed to a reachable CIDR
sudo mount -t nfs <gateway-ip>:/<bucket> /mnt/gw
```

## Notes d'exploitation

- Une file gateway donne un accès NFS/SMB au bucket S3 qui se trouve derrière, sans permissions sur l'API S3, contournant les contrôles de bucket policy qui supposent un accès par API.
- Les cibles iSCSI d'une volume gateway sans CHAP, ou avec un CHAP faible, sont montables par quiconque peut atteindre l'interface réseau de la gateway.
- L'accès au plan de contrôle de la gateway (`storagegateway:*`) permet de créer un nouveau partage sur un bucket existant, une voie de lecture plus discrète que de toucher directement à S3.

## Outils

- **AWS CLI** (`storagegateway list-/describe-*`) : énumérer les gateways, les partages et leurs emplacements sous-jacents.
- **mount.nfs** / **iscsiadm** : monter les partages et volumes exposés.

## Références

- [HackTricks Cloud : Storage Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [AWS : partages de fichiers Storage Gateway](https://docs.aws.amazon.com/storagegateway/latest/userguide/GettingStartedCreateFileShare.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
