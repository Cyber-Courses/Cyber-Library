---
title: "GCP"
description: "Attaquer Google Cloud Platform : énumérer et abuser de Cloud IAM, usurper l'identité de comptes de service, récolter des identifiants depuis les métadonnées et Secret Manager, et exploiter le calcul, le stockage, le serverless, les données, le réseau, la journalisation et Pub/Sub à travers les projets et l'organisation."
keywords:
  - GCP
  - Cloud IAM
  - compte de service
  - actAs
  - projet
---

# GCP

Google Cloud se pilote via les Cloud APIs, autorisées par les liaisons **Cloud IAM** et atteintes avec un jeton utilisateur, une clé de **compte de service** ou le compte de service attaché à une ressource. Presque chaque attaque est une question d'IAM : quels membres détiennent quels rôles à quel scope (organisation, dossier, projet ou ressource), quelle permission vous permet d'**usurper l'identité** ou d'**actAs** un compte de service plus privilégié, et sur quelle ressource vous pouvez déployer pour s'exécuter en tant que tel. Les surfaces ci-dessous suivent cette forme, l'**escalade de privilèges** résidant dans [identity](identity/index.md).

## Énumération

L'énumération se fond dans chaque surface, mais l'inventaire à l'échelle du projet et de l'organisation est exécuté en premier : `gcloud projects list` et `gcloud asset search-all-resources` pour le graphe de ressources, `gcloud projects get-iam-policy` pour les liaisons, et **GCPBucketBrute**, **gcloud** et l'outillage de graphe IAM pour déterminer qui peut atteindre quel compte de service. Ceux-ci alimentent chaque surface ci-dessous.

## Surfaces

- **[Identity](identity/index.md)** : Cloud IAM `setIamPolicy`, le catalogue `actAs` de déploiement en tant que compte de service, les métadonnées, l'usurpation d'identité de compte de service et la fédération d'identité de charge de travail.
- **[Credentials](credentials/index.md)** : le serveur de métadonnées, Secret Manager, les clés de compte de service, les jetons gcloud et ADC, Cloud KMS, les clés HMAC et les clés d'API.
- **[Compute](compute/index.md)** : les métadonnées Compute Engine, SSH et les scopes de compte de service, ainsi que l'accès au cluster GKE et l'identité des nœuds.
- **[Storage](storage/index.md)** : l'énumération et l'accès aux buckets Cloud Storage, les instantanés de disque et Filestore.
- **[Serverless](serverless/index.md)** : l'abus des comptes de service et déclencheurs de Cloud Functions, Cloud Run et Cloud Build.
- **[Data](data/index.md)** : Cloud SQL, BigQuery, Vertex AI, Dataproc, Dataflow, Firestore, Spanner et Bigtable.
- **[Networking](networking/index.md)** : l'exposition par règles de pare-feu, la portée VPC, Cloud DNS et la prise de contrôle de sous-domaine, et l'équilibrage de charge.
- **[Logging and detection](logging-and-detection/index.md)** : la falsification des sinks Cloud Logging, la désactivation des journaux d'audit d'accès aux données et l'affaiblissement de Security Command Center.
- **[Messaging](messaging/index.md)** : les sujets et abonnements Pub/Sub, et les files d'attente Cloud Tasks.

## Références

- [HackTricks Cloud : GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs : escalade de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
