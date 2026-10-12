---
title: "Stockage AWS"
order: 4
description: "Attaquer le stockage AWS : énumération des buckets S3, accès public et abus de politique de bucket et d'ACL, récupération de données EBS et EFS, accès FSx et Storage Gateway, et coffres AWS Backup."
keywords:
  - S3
  - EBS
  - EFS
  - bucket policy
  - snapshots
---

# Stockage

Le stockage, c'est là où résident les données, c'est donc l'objectif habituel une fois qu'un principal est détenu. Le stockage AWS se divise en stockage objet (S3), stockage par blocs (EBS, exposé au travers de ses instantanés), systèmes de fichiers partagés (EFS, FSx), le plan de sauvegarde (AWS Backup) et le pont hybride (Storage Gateway). La plupart des compromissions de stockage ne relèvent pas d'un exploit mais d'un problème de permission ou d'exposition : un bucket lisible par tous, un instantané partagé avec l'ensemble des comptes, une cible de montage accessible depuis un sous-réseau que vous contrôlez.

## Ce qui est regroupé ici

- **[S3](s3/index.md)** : découverte de buckets et d'objets, accès public, et abus de politique de bucket et d'ACL.
- **[Instantanés EBS](ebs-snapshots.md)** : instantanés de stockage par blocs publics ou partagés, restaurés pour lire leurs volumes.
- **[EFS](efs.md)** : systèmes de fichiers NFS exposés, montés via des groupes de sécurité ou une politique de système de fichiers permissifs.
- **[Backup](backup.md)** : points de récupération lus et restaurés depuis les coffres AWS Backup.
- **[FSx](fsx.md)** : systèmes de fichiers Windows, Lustre et NetApp atteints via les permissions de partage.
- **[Storage Gateway](storage-gateway.md)** : partages de fichiers et volumes en cache faisant le pont entre l'accès sur site et S3 et EBS.

L'énumération est intégrée à chaque page : trouver la ressource est la première moitié du travail pour atteindre ses données.

## Références

- [HackTricks Cloud : AWS S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Rhino Security Labs : tests d'intrusion du stockage AWS](https://rhinosecuritylabs.com/aws/penetration-testing-aws-storage/)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
