---
title: "Données GCP"
order: 6
description: "Attaquer les services de données GCP : Cloud SQL, BigQuery, Vertex AI, Dataproc, Dataflow, Firestore, Spanner et Bigtable."
keywords:
  - Cloud SQL
  - BigQuery
  - Vertex AI
  - Dataproc
  - Firestore
  - data
---

# Données

Les magasins de données managés sont l'endroit où un engagement GCP porte ses fruits, et les atteindre se ramène à deux mouvements qui reviennent dans chaque service traité ici. Soit le **plan de données** est exposé directement (une IP publique, un réseau autorisé trop large, une liaison `roles/*.dataViewer` trop permissive), soit un **service s'exécute sous un compte de service attaché** que vous pouvez piloter, et le job devient alors à la fois du calcul et un chemin vers le jeton de ce compte. Les services d'analytique et de ML en particulier font aussi office de calcul et sont référencés de manière croisée dans [identity](../identity/index.md), là où le compte de service est le butin.

## Ce qui est regroupé ici

- **[Cloud SQL](cloud-sql.md)** : le SQL Auth Proxy, l'IP publique et les réseaux autorisés, et les utilisateurs de base de données intégrés.
- **[BigQuery](bigquery.md)** : lecture des datasets, exfiltration par requête et export, et IAM des datasets.
- **[Vertex AI](vertex-ai.md)** : comptes de service des notebooks et des jobs d'entraînement, et vol de modèles et de datasets.
- **[Dataproc](dataproc.md)** : le compte de service du cluster, la soumission de jobs et les données stagées.
- **[Dataflow](dataflow.md)** : le compte de service worker et les données sources et de destination du pipeline.
- **[Firestore](firestore.md)** : les collections via un IAM de base de données trop large ou des failles de règles de sécurité.
- **[Spanner](spanner.md)** : les bases de données via l'accès `spanner.databases`.
- **[Bigtable](bigtable.md)** : les tables via l'accès `bigtable.tables`.

## Références

- [HackTricks Cloud: GCP databases](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Datadog Security Labs: cloud attack research](https://securitylabs.datadoghq.com/)
- [Google Cloud: IAM for data services](https://cloud.google.com/iam/docs/)
