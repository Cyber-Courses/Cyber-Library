---
title: "Access: reaching live RDS instances through exposure and weak auth"
description: "Reaching RDS instances through publicly-exposed endpoints, weak credentials, or RDS IAM authentication."
keywords:
  - RDS
  - endpoint
  - IAM authentication
  - database credentials
  - access
---

# Access

A running RDS instance is reachable whenever its endpoint is publicly accessible and its security group allows your source, or when you can mint credentials for it. The master and application credentials frequently sit in [Secrets Manager](../../credentials/secret-stores/secrets-manager.md) or an EC2 user-data script, and when RDS IAM authentication is enabled, a principal with `rds-db:connect` mints a short-lived token with no password at all.

## Finding reachable instances

```bash
aws rds describe-db-instances \
  --query 'DBInstances[?PubliclyAccessible==`true`].[DBInstanceIdentifier,Endpoint.Address,Endpoint.Port]'
```

## Connecting

```bash
# IAM authentication: no password, mint a token
TOKEN=$(aws rds generate-db-auth-token --hostname <ep> --port 3306 --username iam_user)
mysql -h <ep> -P 3306 -u iam_user --password="$TOKEN" --enable-cleartext-plugin

# or with credentials recovered from Secrets Manager / user-data
psql "host=<ep> port=5432 dbname=app user=admin password=<pw>"
```

## Exploitation notes

- `PubliclyAccessible` plus a security group open to `0.0.0.0/0` is the common exposure; even without it, an instance is reachable from a compromised EC2 box in the same VPC.
- RDS IAM auth tokens last 15 minutes but can be re-minted indefinitely while the permission holds.
- Recovered master credentials also allow a password change and durable access; weigh that against the noise of altering the account.

## Tools

- **AWS CLI** (`rds describe-db-instances`, `generate-db-auth-token`).
- **mysql / psql / sqlcmd**: native clients once reachable.

## References

- [AWS: IAM database authentication](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)
- [HackTricks Cloud: AWS RDS access](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable RDS instances](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
