---
title: "Clés HMAC : générer une clé HMAC Cloud Storage pour un compte de service"
order: 6
description: "Créer une clé HMAC Cloud Storage pour un compte de service avec storage.hmacKeys.create afin d'accéder aux buckets en tant que ce compte."
keywords:
  - clés HMAC
  - storage.hmacKeys.create
  - Cloud Storage
  - interopérabilité
  - compte de service
  - API S3
---

# Clés HMAC

Cloud Storage prend en charge des **clés HMAC** compatibles S3 pour l'interopérabilité. `storage.hmacKeys.create` permet de générer une paire clé d'accès/secret liée à un compte de service, paire qui donne ensuite accès à tous les buckets accessibles par ce compte, via l'API XML (S3), en dehors du chemin OAuth habituel et de sa journalisation. C'est à la fois un mouvement latéral (agir en tant que le compte de service sur le stockage) et un identifiant durable et peu visible.

## Créer et utiliser une clé

```bash
# mint an HMAC key for a service account you can act for
gsutil hmac create <sa>@<project>.iam.gserviceaccount.com
# -> Access ID and Secret

# use it with any S3 client against the GCS XML endpoint
AWS_ACCESS_KEY_ID=<accessid> AWS_SECRET_ACCESS_KEY=<secret> \
  aws --endpoint-url https://storage.googleapis.com s3 ls
```

## Notes d'exploitation

- La clé hérite de l'accès au stockage du compte de service : il convient donc de cibler un compte disposant de rôles étendus sur les buckets.
- L'accès HMAC transite par l'API de type S3, que certains outils de supervision et les VPC Service Controls traitent différemment des appels GCS natifs, ce qui en fait un chemin plus discret.
- Les clés persistent jusqu'à suppression explicite, indépendamment des jetons du compte, ce qui en fait aussi un moyen de persistance.

## Outils

- **gsutil** (`hmac create`, `hmac list`).
- Tout **client S3** (`aws s3`, boto3) pointé vers `storage.googleapis.com`.

## Références

- [Google : managing HMAC keys](https://cloud.google.com/storage/docs/authentication/managing-hmackeys)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
