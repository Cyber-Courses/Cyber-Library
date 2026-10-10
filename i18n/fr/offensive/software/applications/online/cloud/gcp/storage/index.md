---
title: "Stockage GCP"
order: 4
description: "Attaquer le stockage GCP : énumération des buckets Cloud Storage et accès IAM et ACL, instantanés de disque persistant, et partages Filestore."
keywords:
  - Cloud Storage
  - bucket
  - instantané de disque
  - Filestore
  - IAM
---

# Stockage

GCP conserve les données en masse dans des buckets Cloud Storage, dans les disques persistants derrière Compute Engine, et dans des partages NFS Filestore. Chacun se rejoint différemment : les buckets par un nom global associé à une IAM ou des ACL héritées, les disques par instantané puis rattachement, et Filestore en montant le partage depuis l'intérieur du VPC. Le travail consiste à trouver le stockage auquel on peut accéder et à en extraire les identifiants, les clés et les bases de données.

## Ce qui est regroupé ici

- **[Cloud Storage](cloud-storage/index.md)** : énumérer les buckets publics et mal configurés, et abuser des IAM et ACL de bucket.
- **[Instantanés de disque](disk-snapshots.md)** : copier un disque persistant et le rattacher pour lire le contenu d'une autre instance.
- **[Filestore](filestore.md)** : monter des partages NFS exposés au sein du VPC.

Les clés HMAC qui accordent un accès interopérable à S3 pour Cloud Storage sont des identifiants et se trouvent sous [identifiants](../credentials/hmac-keys.md).

## Références

- [HackTricks Cloud: GCP storage](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
