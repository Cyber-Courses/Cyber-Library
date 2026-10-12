---
title: "Déploiement actAs"
description: "Déployer une ressource qui s'exécute sous une identité de compte de service privilégié via iam.serviceAccounts.actAs : création d'une instance Compute, d'une fonction, d'un service Cloud Run, d'un build ou d'un job lié à un compte à privilèges supérieurs."
keywords:
  - actAs
  - service account
  - deploy
  - privilege escalation
  - compute.instances.create
  - Cloud Build
---

# Déploiement actAs

`iam.serviceAccounts.actAs` est la permission qui permet de lier un compte de service à une ressource que vous créez. Seule, elle ne fait rien ; combinée à une permission de création de ressource, elle vous permet de déployer un élément qui **s'exécute sous l'identité** d'un compte de service plus privilégié que vous, puis de récupérer son jeton ou d'exécuter du code en son nom. Le compte de service cible a seulement besoin d'exister et d'être utilisable dans le projet.

Le schéma est toujours le même : trouver un compte de service privilégié (`gcloud iam service-accounts list`, et le graphe d'[énumération](../../enumeration.md) pour ses liaisons de rôles), confirmer que vous détenez `actAs` sur lui, puis le piloter via l'un des déploiements ci-dessous.

## Cibles de déploiement

- **[Instance Compute](compute-instance.md)** : `compute.instances.create` avec un compte de service privilégié et un scope complet.
- **[Cloud Function](cloud-function.md)** : déployer une fonction s'exécutant sous le compte attaché.
- **[Cloud Run](cloud-run.md)** : un service ou un job exécutant des conteneurs de l'attaquant sous l'identité du compte.
- **[Deployment Manager](deployment-manager.md)** : déployer sous l'agent de service des API Google (Éditeur par défaut).
- **[Cloud Build](cloud-build.md)** : exfiltrer le jeton du compte de service Cloud Build en cours de build.
- **[App Engine](app-engine.md)** : une version s'exécutant sous le compte par défaut App Engine.
- **[Cloud Composer](cloud-composer.md)** : une tâche de DAG Airflow s'exécutant sous le compte de l'environnement.
- **[Cloud Scheduler](cloud-scheduler.md)** : un job planifié appelant une API sous l'identité d'un compte choisi.
- **[Notebook Vertex AI](vertex-ai-notebook.md)** : une VM de notebook avec le compte attaché.
- **[Dataproc](dataproc.md)** : un cluster ou un job s'exécutant sous le compte du cluster.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : iam.serviceAccounts.actAs](https://cloud.google.com/iam/docs/service-accounts-actas)
