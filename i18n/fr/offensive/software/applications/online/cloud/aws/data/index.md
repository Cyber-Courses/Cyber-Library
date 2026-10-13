---
title: "Données AWS"
order: 6
description: "Attaquer les services de données AWS : snapshots et accès RDS, DynamoDB, Redshift, SageMaker, Glue, Athena, Lake Formation, DocumentDB, ElastiCache, EMR, Neptune, Timestream et Keyspaces."
keywords:
  - RDS
  - DynamoDB
  - Redshift
  - SageMaker
  - Glue
  - Athena
  - Lake Formation
---

# Données

Les magasins de données managés sont là où l'engagement porte ses fruits : la base de données, l'entrepôt, le lac et le cache contiennent ce que le compte existe pour protéger. Les atteindre se décline en deux manœuvres qui reviennent à travers tous les services traités ici. Soit le **plan de données** est exposé directement (point de terminaison accessible publiquement, identifiants faibles ou partagés, autorisation trop large), soit le **plan de contrôle** vous livre les données hors ligne (snapshot partagé, export vers S3, appel de distribution d'identifiants). Plusieurs de ces services exécutent aussi des jobs sous un rôle attaché, de sorte qu'ils font également office de calcul et sont recoupés avec [identity](../identity/index.md) lorsque ce rôle est le prix.

## Ce qui est regroupé ici

- **[RDS](rds/index.md)** : partage et restauration de snapshots, et atteindre l'instance via des contrôles réseau et d'authentification faibles.
- **[DynamoDB](dynamodb.md)** : `Scan`, `Query` et export de table vers S3.
- **[SageMaker](sagemaker.md)** : notebooks, jobs d'entraînement et endpoints, ainsi que le rôle attaché.
- **[Redshift](redshift.md)** : clusters exposés, identifiants temporaires et autorisations de base de données.
- **[Glue](glue.md)** : le Data Catalog, les scripts de jobs et l'exécution de jobs sous le rôle Glue.
- **[DocumentDB](documentdb.md)** : clusters compatibles Mongo exposés.
- **[ElastiCache](elasticache.md)** : Redis et Memcached non authentifiés.
- **[Athena](athena.md)** : interroger des données S3 via le catalogue.
- **[Lake Formation](lake-formation.md)** : permissions des tables gouvernées et distribution d'identifiants.
- **[EMR](emr.md)** : clusters, étapes et rôle d'instance.
- **[Neptune](neptune.md)** : clusters de base de données graphe exposés.
- **[Timestream](timestream.md)** : interroger des bases de données de séries temporelles.
- **[Keyspaces](keyspaces.md)** : tables compatibles Cassandra.

## Références

- [HackTricks Cloud: AWS databases](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: security in Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.html)
- [CloudFox: enumerating exploitable cloud data services](https://github.com/BishopFox/cloudfox)
- [Prowler: AWS data-exposure and misconfiguration checks](https://github.com/prowler-cloud/prowler)
