---
title: "Énumération : découvrir les buckets GCP et les objets lisibles"
order: 1
description: "Trouver des buckets GCP et des objets lisibles par devinette de nom et listage, avec des outils comme GCPBucketBrute."
keywords:
  - énumération de bucket
  - GCPBucketBrute
  - bucket public
  - listage d'objets
  - GCS
---

# Énumération

Les noms de buckets GCS sont globaux, donc un nom existe ou n'existe pas, et l'existence est vérifiable sans aucune identification. L'énumération consiste à deviner des noms à partir de l'image de marque de la cible, à les vérifier, puis à lister ce qu'un appelant anonyme ou votre identité actuelle peut lire.

## Découverte non authentifiée

```bash
# a bucket's JSON API endpoint distinguishes 200/403 (exists) from 404 (absent)
curl -s -o /dev/null -w '%{http_code}\n' \
  "https://storage.googleapis.com/storage/v1/b/target-backups"

# anonymous object list, if the bucket allows it
gcloud storage ls gs://target-backups --no-user-output-enabled 2>/dev/null
gsutil ls gs://target-backups
```

## Deviner des noms à grande échelle

```bash
# GCPBucketBrute: permute a keyword, test existence and the ACL/IAM exposure,
# and flag buckets that allow privilege escalation (setIamPolicy by allUsers)
python3 gcpbucketbrute.py -k target -u          # -u = unauthenticated
cloud_enum -k target -k target-prod             # GCS + S3 + Azure in one pass
```

## Listage authentifié

```bash
gcloud storage buckets list --project <proj>
gcloud storage ls --recursive gs://<bucket>
gsutil iam get gs://<bucket>                    # read the bucket IAM policy
```

## Notes d'exploitation

- GCPBucketBrute signale les buckets dont l'IAM permet à `allUsers` d'appeler `storage.buckets.setIamPolicy`, ce qui constitue une prise de contrôle directe du bucket.
- Un `403` est un résultat positif : le bucket existe et mérite d'être attaqué par IAM ou ACL même lorsque le listage est refusé.
- Les noms d'objets divulguent la structure ; demandez directement `terraform.tfstate`, `.env`, et des objets de sauvegarde datés, même sans permission de listage.

## Outils

- **GCPBucketBrute** : existence, exposition ACL/IAM, et détection d'élévation de privilèges à partir d'un mot-clé.
- **cloud_enum** : découverte de buckets publics multi-fournisseurs.
- **gsutil / gcloud storage** : listage anonyme et authentifié.

## Références

- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [cloud_enum (initstring)](https://github.com/initstring/cloud_enum)
- [HackTricks Cloud: GCP buckets enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
