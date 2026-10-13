---
title: "Timestream: querying time-series databases"
order: 12
description: "Querying Timestream time-series databases that the caller should not have access to."
keywords:
  - Timestream
  - time series
  - query
  - database
  - data access
---

# Timestream

Timestream is AWS's time-series database, accessed entirely through IAM with no network surface. A principal holding `timestream:Select` queries any database and table the policy allows, which is frequently broader than intended because Timestream permissions are often granted at the service level. The data, operational and IoT telemetry, can reveal infrastructure, behaviour, and volumes useful for the wider engagement.

## Querying

```bash
aws timestream-write list-databases
aws timestream-write list-tables --database-name <db>
aws timestream-query query \
  --query-string 'SELECT * FROM "db"."table" LIMIT 100'
```

## Exploitation notes

- Query endpoints are discovered with `timestream-query describe-endpoints`; the SDK handles this, but note it when calling the API directly.
- Service-level `timestream:*` grants are common and give read across every database in the account.
- The data is append-only telemetry, so the value is intelligence rather than tampering.

## Tools

- **AWS CLI** (`timestream-query query`, `timestream-write list-databases`).

## References

- [AWS: Timestream access control](https://docs.aws.amazon.com/timestream/latest/developerguide/security-iam.html)
- [HackTricks Cloud: AWS Timestream](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
