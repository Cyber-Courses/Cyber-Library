---
title: "Bigtable : lire des tables via l'accès bigtable.tables"
order: 8
description: "Attaquer Cloud Bigtable : lire des tables via l'accès bigtable.tables avec des rôles d'instance étendus."
keywords:
  - Bigtable
  - table
  - bigtable.tables
  - instance
  - NoSQL
  - accès aux données
---

# Bigtable

Cloud Bigtable est un magasin NoSQL en colonnes larges accessible via IAM. Un principal disposant de `bigtable.tables.readRows` (ou d'un rôle d'instance étendu comme `roles/bigtable.reader`) lit toute table de l'instance, l'attaque consiste donc à énumérer les instances et tables atteignables et à en extraire les lignes.

## Lire une table

```bash
gcloud bigtable instances list
cbt -instance <inst> ls                 # list tables
cbt -instance <inst> read <table> count=1000
```

## Notes d'exploitation

- Les rôles au niveau de l'instance s'appliquent à chaque table ; une seule liaison étendue expose l'instance entière.
- Les rôles en écriture (`bigtable.tables.mutateRows`) permettent la falsification de lignes lorsque les données sous-tendent l'état de l'application.
- Bigtable n'a pas de point de terminaison public ; l'accès repose sur la liaison IAM de votre principal ou d'un compte de service usurpé.

## Outils

- **cbt** / **gcloud** (`bigtable instances list`, `cbt read`).
- **Bibliothèques clientes Bigtable** pour des analyses scriptées.

## Références

- [HackTricks Cloud : GCP Bigtable](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google Cloud : contrôle d'accès Bigtable](https://cloud.google.com/bigtable/docs/access-control)
