---
title: "Spanner : lire des bases de données via l'accès spanner.databases"
order: 7
description: "Attaquer Cloud Spanner : lire des bases de données via l'accès spanner.databases et l'abus de sessions."
keywords:
  - Cloud Spanner
  - database
  - spanner.databases
  - session
  - SQL
  - data access
---

# Spanner

Cloud Spanner est une base de données relationnelle distribuée à l'échelle mondiale, accessible via IAM. Un principal détenant `spanner.databases.read` (ou `roles/spanner.databaseReader`) ouvre une session et interroge n'importe quelle table de la base de données ; l'attaque consiste donc simplement à énumérer les instances et bases de données qu'un jeton peut atteindre, puis à les lire.

## Lire une base de données

```bash
gcloud spanner instances list
gcloud spanner databases list --instance <inst>
gcloud spanner databases execute-sql <db> --instance <inst> \
  --sql='SELECT * FROM Users LIMIT 1000'
```

## Notes d'exploitation

- `spanner.databases.beginOrRollbackReadWriteTransaction` et les permissions d'écriture associées permettent d'altérer des lignes là où le rôle est trop large.
- Spanner n'a aucune surface réseau publique ; l'accès repose donc uniquement sur la liaison IAM de votre principal ou d'un compte de service usurpé. Poursuivez la liaison plutôt que le réseau.

## Outils

- **gcloud** (`spanner databases execute-sql`).
- **Spanner client libraries** pour des lectures de session scriptées.

## Références

- [HackTricks Cloud: GCP Spanner](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Spanner IAM](https://cloud.google.com/spanner/docs/iam)
