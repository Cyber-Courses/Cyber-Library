---
title: "Lake Formation : distribution d'identifiants pour les tables gouvernées"
order: 9
description: "Abuser des permissions Lake Formation et de la distribution d'identifiants pour lire des tables gouvernées du lac de données."
keywords:
  - Lake Formation
  - lac de données
  - permissions
  - distribution d'identifiants
  - table
---

# Lake Formation

Lake Formation gouverne l'accès aux tables du lac de données et distribue des identifiants S3 éphémères, limités à ce qu'un principal s'est vu accorder. L'attaque consiste à détenir ou à s'octroyer soi-même une permission Lake Formation, puis à appeler l'API de distribution, qui renvoie des identifiants temporaires pour les données S3 sous-jacentes, contournant la stratégie propre au compartiment. Un principal capable d'administrer les autorisations Lake Formation peut étendre son propre accès à n'importe quelle table gouvernée.

## Distribuer des identifiants pour une table

```bash
aws lakeformation list-permissions
aws lakeformation get-temporary-glue-table-credentials \
  --table-arn <table-arn> --supported-permission-types COLUMN_PERMISSION
# the response contains temporary S3 credentials for the governed data
```

## S'octroyer l'accès

```bash
aws lakeformation grant-permissions \
  --principal DataLakePrincipalIdentifier=<your-arn> \
  --resource '{"Table":{"DatabaseName":"db","Name":"t"}}' \
  --permissions SELECT
```

## Notes d'exploitation

- La distribution d'identifiants renvoie de vrais identifiants S3 pour l'emplacement gouverné, de sorte qu'elle lit les données même lorsque la stratégie de compartiment refuserait directement l'appelant.
- `lakeformation:GrantPermissions` avec une portée d'administration est une élévation de privilèges au plan de données : octroyez, puis distribuez.
- Les tables gouvernées protègent souvent les données les plus sensibles du lac, ce qui justifie l'étape supplémentaire de la voie de distribution.

## Outils

- **AWS CLI** (`lakeformation get-temporary-glue-table-credentials`, `grant-permissions`).

## Références

- [AWS: Lake Formation credential vending](https://docs.aws.amazon.com/lake-formation/latest/dg/credential-vending.html)
- [HackTricks Cloud: AWS Lake Formation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating exploitable cloud data services](https://github.com/BishopFox/cloudfox)
- [Datadog Security Labs: AWS attack research](https://securitylabs.datadoghq.com/)
