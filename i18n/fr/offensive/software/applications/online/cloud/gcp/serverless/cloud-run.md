---
title: "Cloud Run : services non authentifiés, révisions et le compte de service attaché"
order: 2
description: "Attaquer les services et jobs Cloud Run : invocation non authentifiée, secrets d'environnement et de révision, et le compte de service d'exécution attaché."
keywords:
  - Cloud Run
  - compte de service d'exécution
  - invocation
  - révision
  - environnement
---

# Cloud Run

Cloud Run exécute un conteneur avec un **compte de service** attaché. Les services exposent un point de terminaison HTTPS (public en option), et chaque **révision** porte son environnement et ses références de secrets. L'accès en lecture les divulgue ; l'accès au déploiement exécute votre conteneur avec le compte de service.

## Lire le service, les révisions et l'environnement

```bash
gcloud run services describe <svc> --region <r> \
  --format='value(spec.template.spec.serviceAccountName)'
gcloud run services describe <svc> --region <r> --format=export   # env vars, secret refs, image
gcloud run revisions list --service <svc> --region <r>
```

## Invocation non authentifiée

```bash
# a service with the allUsers invoker binding is reachable without creds
gcloud run services get-iam-policy <svc> --region <r>
curl -s https://<svc>-<hash>-<region>.a.run.app/
```

## Exécuter en tant que compte de service attaché

```bash
gcloud run deploy <svc> --region <r> --image <you>/img \
  --service-account <privileged-sa>@<proj>.iam.gserviceaccount.com \
  --allow-unauthenticated
# the container reads the SA token from the metadata server, as in Cloud Functions
```

## Notes d'exploitation

- Le déploiement nécessite `iam.serviceAccounts.actAs` sur le compte de service attaché, ce qui en fait un chemin d'élévation de privilèges lorsque ce compte dispose de droits supérieurs aux vôtres.
- Les **jobs** Cloud Run (`gcloud run jobs create/execute`) fonctionnent de la même manière sans point de terminaison HTTP, utile pour une tâche ponctuelle sous le compte de service.
- Les références Secret Manager dans une révision se résolvent à l'exécution ; extraire la configuration de la révision ainsi que l'environnement du conteneur permet de les récupérer.

## Outils

- **gcloud** (`run deploy`, `services describe`, `run jobs`, `get-iam-policy`).

## Références

- [HackTricks Cloud: GCP Cloud Run](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Cloud Run service identity](https://cloud.google.com/run/docs/securing/service-identity)
