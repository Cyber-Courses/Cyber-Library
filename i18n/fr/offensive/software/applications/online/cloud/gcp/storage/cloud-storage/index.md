---
title: "Cloud Storage"
description: "Attaquer les buckets Cloud Storage : énumérer les buckets publics et mal configurés, et abuser des IAM et ACL de bucket pour lire ou écrire des objets."
keywords:
  - Cloud Storage
  - GCS
  - bucket
  - IAM
  - ACL
---

# Cloud Storage

Les buckets Cloud Storage (GCS) partagent un espace de noms global unique, ce qui les rend détectables sans identification, et l'accès est décidé soit par l'**IAM de bucket** (uniforme), soit par des **ACL héritées** (granulaires). Les deux modes de défaillance sont un bucket exposé à `allUsers`/`allAuthenticatedUsers` et une liaison IAM trop permissive (`roles/storage.admin` ou `objectViewer` sur le projet), qui livrent tous deux les objets qu'il contient à un attaquant.

## Ce qui est regroupé ici

- **[Énumération](enumeration.md)** : trouver des buckets et des objets lisibles par devinette de nom et listage.
- **[Accès](access.md)** : lire ou écrire des objets via des IAM de bucket, des ACL, ou des liaisons anonymes trop permissives.

## Références

- [HackTricks Cloud: GCP buckets](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [Google: Cloud Storage access control](https://cloud.google.com/storage/docs/access-control)
