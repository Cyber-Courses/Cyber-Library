---
title: "Azure Blob storage"
order: 1
description: "Attaquer Azure Blob storage : trouver des conteneurs publics et lire des blobs via des clés de compte, le RBAC ou l'accès anonyme."
keywords:
  - blob storage
  - container
  - anonymous access
  - account key
  - public blob
---

# Blob storage

Azure Blob storage conserve des objets dans des **conteneurs** à l'intérieur d'un **compte de stockage**, adressés à `https://<account>.blob.core.windows.net/<container>`. L'accès se fait de trois façons, et chacune est un chemin d'attaque : **anonyme** lorsque le niveau d'accès public d'un conteneur est réglé sur blob ou container, une **clé de compte** (le credential maître de l'ensemble du compte), ou des rôles de données **Azure RBAC** tels que Storage Blob Data Reader. Le travail consiste à trouver le compte, à trouver un conteneur, et à atteindre ses blobs par celle des trois voies qui est ouverte.

## Ce qui se rattache ici

- **[Énumération](enumeration.md)** : découvrir les comptes de stockage et les conteneurs publics par devinette de noms et listage anonyme.
- **[Accès](access.md)** : lire et écrire des blobs via des clés de compte, des rôles de données RBAC ou un accès anonyme mal configuré.

## Références

- [HackTricks Cloud: Azure storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-storage.html)
- [NetSPI: anonymous blob access and MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: blob public access levels](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)
