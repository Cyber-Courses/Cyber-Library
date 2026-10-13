---
title: "Neptune: reaching graph-database clusters"
order: 11
description: "Reaching Neptune graph-database clusters through exposed endpoints and IAM authentication gaps."
keywords:
  - Neptune
  - graph database
  - cluster
  - endpoint
  - IAM auth
---

# Neptune

Neptune is AWS's managed graph database, reached over its cluster endpoint inside a VPC. Where IAM database authentication is disabled, any principal that can reach the endpoint queries it with no further auth; where it is enabled, a SigV4-signed request from a holding principal is accepted. The graph often encodes identity and relationship data that is valuable in its own right.

## Finding and querying

```bash
aws neptune describe-db-clusters \
  --query 'DBClusters[].[DBClusterIdentifier,Endpoint,Port,IAMDatabaseAuthenticationEnabled]'

# Gremlin over HTTP against a reachable endpoint (IAM auth disabled)
curl -s https://<endpoint>:8182/gremlin \
  -d '{"gremlin":"g.V().limit(100)"}'
```

## Exploitation notes

- Neptune lives in a VPC, so this is reached from a compromised EC2 host or a pivot, not the internet.
- With IAM auth off, endpoint reachability is the only control; with it on, a signed request from a permitted principal still works.
- Both Gremlin and SPARQL endpoints are exposed depending on the engine, so try both.

## Tools

- **AWS CLI** (`neptune describe-db-clusters`).
- **curl / gremlin-console / SPARQL clients**: native query access.

## References

- [AWS: Neptune IAM authentication](https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth.html)
- [HackTricks Cloud: AWS Neptune](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
