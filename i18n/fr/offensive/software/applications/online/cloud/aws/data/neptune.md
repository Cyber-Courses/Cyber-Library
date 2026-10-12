---
title: "Neptune : atteindre des clusters de base de données graphe"
order: 11
description: "Atteindre des clusters de base de données graphe Neptune via des points de terminaison exposés et des lacunes d'authentification IAM."
keywords:
  - Neptune
  - base de données graphe
  - cluster
  - point de terminaison
  - authentification IAM
---

# Neptune

Neptune est la base de données graphe managée d'AWS, atteinte via son point de terminaison de cluster à l'intérieur d'un VPC. Lorsque l'authentification de base de données IAM est désactivée, tout principal capable d'atteindre le point de terminaison l'interroge sans authentification supplémentaire ; lorsqu'elle est activée, une requête signée en SigV4 par un principal détenteur est acceptée. Le graphe encode souvent des données d'identité et de relations qui ont de la valeur en soi.

## Trouver et interroger

```bash
aws neptune describe-db-clusters \
  --query 'DBClusters[].[DBClusterIdentifier,Endpoint,Port,IAMDatabaseAuthenticationEnabled]'

# Gremlin over HTTP against a reachable endpoint (IAM auth disabled)
curl -s https://<endpoint>:8182/gremlin \
  -d '{"gremlin":"g.V().limit(100)"}'
```

## Notes d'exploitation

- Neptune vit dans un VPC, donc on l'atteint depuis un hôte EC2 compromis ou par un pivot, pas depuis Internet.
- Avec l'authentification IAM désactivée, la joignabilité du point de terminaison est le seul contrôle ; avec elle activée, une requête signée par un principal autorisé fonctionne quand même.
- Les points de terminaison Gremlin et SPARQL sont exposés selon le moteur, essayez donc les deux.

## Outils

- **AWS CLI** (`neptune describe-db-clusters`).
- **curl / gremlin-console / clients SPARQL** : accès natif aux requêtes.

## Références

- [AWS: Neptune IAM authentication](https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth.html)
- [HackTricks Cloud: AWS Neptune](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
