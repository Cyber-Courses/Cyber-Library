---
title: "Données Azure"
order: 6
description: "Attaquer les services de données Azure : accès à SQL Database, clés Cosmos DB, pipelines Data Factory, Synapse Analytics et tables de stockage."
keywords:
  - données Azure
  - SQL Database
  - Cosmos DB
  - Data Factory
  - Synapse
---

# Données

Les services de données managés d'Azure s'atteignent de deux façons : par les clés ou chaînes de connexion propres au service (clés Cosmos DB, clés de compte de stockage), ou par un principal dont le RBAC Azure et l'identité de base de données lui permettent de se connecter. Les services d'analytique sont différents : Data Factory et Synapse exécutent des pipelines sous leur propre **identité managée**, de sorte que compromettre l'un d'eux est une voie vers le jeton de cette identité, pas seulement vers les données.

## Pages

- **[SQL Database](sql-database/index.md)** : authentification Entra et SQL, portée du pare-feu et identité managée du serveur.
- **[Cosmos DB](cosmos-db.md)** : clés primaires et en lecture seule, resource tokens et accès aux documents par RBAC.
- **[Data Factory](data-factory.md)** : pipelines et linked services qui s'exécutent sous l'identité managée de la factory.
- **[Synapse Analytics](synapse-analytics.md)** : pools SQL et Spark, pipelines et identité managée de l'espace de travail.
- **[Tables de stockage](storage-tables.md)** : lecture de Table storage via les clés de compte ou un SAS.

## Références

- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : documentation des services de données Azure](https://learn.microsoft.com/azure/)
