---
title: "Azure storage"
order: 4
description: "Attaquer le stockage Azure : énumération et accès aux conteneurs de blobs, abus de jetons SAS, vol de snapshots de disque et partages Azure Files."
keywords:
  - Azure storage
  - blob
  - SAS token
  - disk snapshot
  - file share
---

# Storage

Le stockage est l'endroit où résident les données : c'est donc l'objectif habituel une fois une prise de pied ou une clé de stockage détenue. Le stockage Azure se décompose en stockage d'objets (**Blob**, exposé via des conteneurs publics, des clés de compte et des SAS), stockage de blocs (**snapshots de disque** managé restaurés hors ligne) et systèmes de fichiers partagés (**Azure Files** via SMB). La plupart des compromissions ici relèvent de l'exposition ou du vol de clé plutôt que d'un exploit : un conteneur anonyme, une URL SAS divulguée valable un an, un snapshot exporté vers un disque que vous contrôlez.

## Ce qui se rattache ici

- **[Blob storage](blob-storage/index.md)** : trouver des conteneurs publics et lire des blobs via des clés de compte, le RBAC ou l'accès anonyme.
- **[Jetons SAS](sas-tokens.md)** : signatures d'accès partagé (account, service et user-delegation) sur-privilégiées.
- **[Snapshots de disque](disk-snapshots.md)** : prendre un snapshot et exporter un disque managé pour lire le disque OS d'une VM hors ligne.
- **[Partages de fichiers](file-shares.md)** : partages SMB Azure Files atteints via des clés de stockage ou un accès basé sur l'identité.

L'énumération se rattache à chaque page : trouver le compte ou le partage est la première moitié de l'accès à ses données. La clé de compte, lorsqu'elle est récupérée, est le credential maître de tous les plans de données ci-dessous et est traitée sous [credentials](../credentials/index.md).

## Références

- [HackTricks Cloud: Azure Blob storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: authorizing access to blob data](https://learn.microsoft.com/azure/storage/common/authorize-data-access)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
