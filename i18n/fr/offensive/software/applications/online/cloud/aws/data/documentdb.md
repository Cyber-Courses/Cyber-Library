---
title: "DocumentDB : atteindre des clusters compatibles Mongo via l'exposition"
order: 6
description: "Atteindre des clusters DocumentDB via des points de terminaison exposés et des identifiants faibles pour lire des données compatibles Mongo."
keywords:
  - DocumentDB
  - MongoDB
  - cluster
  - point de terminaison
  - base de données
---

# DocumentDB

DocumentDB est la base de données compatible MongoDB d'AWS. Elle n'a pas de plan de données IAM : l'accès repose sur le point de terminaison du cluster plus un nom d'utilisateur et un mot de passe de base de données, de sorte qu'elle relève du même schéma d'exposition que Mongo auto-hébergé. Les identifiants résident généralement dans [Secrets Manager](../credentials/secret-stores/secrets-manager.md) ou dans la configuration d'une application, et un cluster joignable depuis un hôte VPC compromis n'est qu'à un identifiant d'une lecture complète.

## Trouver et se connecter

```bash
aws docdb describe-db-clusters \
  --query 'DBClusters[].[DBClusterIdentifier,Endpoint,Port]'
# connect with the mongo shell (TLS on by default)
mongosh "mongodb://<user>:<pw>@<endpoint>:27017/?tls=true&tlsCAFile=global-bundle.pem"
```

## Notes d'exploitation

- DocumentDB se trouve presque toujours à l'intérieur d'un VPC, c'est donc une technique post-ancrage atteinte depuis une machine EC2 ou par un pivot, pas depuis Internet.
- Les identifiants récupérés depuis Secrets Manager ou un serveur d'application sont la clé habituelle ; la base de données elle-même n'a pas de second facteur.
- Videz les collections avec `mongodump` une fois connecté pour une analyse hors ligne.

## Outils

- **AWS CLI** (`docdb describe-db-clusters`).
- **mongosh / mongodump** : clients MongoDB natifs.

## Références

- [AWS: connecting to a DocumentDB cluster](https://docs.aws.amazon.com/documentdb/latest/developerguide/connect.html)
- [HackTricks Cloud: AWS DocumentDB](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
