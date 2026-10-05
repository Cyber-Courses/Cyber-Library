---
title: "ElastiCache: reading unauthenticated Redis and Memcached"
description: "Accessing unauthenticated or exposed Redis and Memcached ElastiCache clusters to read cached data."
keywords:
  - ElastiCache
  - Redis
  - Memcached
  - cluster
  - cache
---

# ElastiCache

ElastiCache runs Redis and Memcached, which ship with no authentication by default. A cluster reachable from a compromised VPC host is usually readable with no credentials at all, and the cache often holds sessions, tokens, and query results that are more immediately useful than the database behind it.

## Finding and reading

```bash
aws elasticache describe-cache-clusters --show-cache-node-info \
  --query 'CacheClusters[].[CacheClusterId,Engine,CacheNodes[].Endpoint.Address]'

# Redis with no AUTH configured
redis-cli -h <endpoint> -p 6379
> KEYS *
> GET session:<id>

# Memcached
echo -e 'stats items\nquit' | nc <endpoint> 11211
```

## Exploitation notes

- Redis AUTH and in-transit TLS are off unless explicitly enabled, so a reachable endpoint is usually a direct read.
- Session and token caches are the high-value keys: a cached session often replays straight into the application as another user.
- Where Redis is writable, poisoning cached authz decisions is an application-level escalation.

## Tools

- **AWS CLI** (`elasticache describe-cache-clusters`).
- **redis-cli / nc**: native access once reachable.

## References

- [AWS: ElastiCache in-transit encryption and AUTH](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/encryption.html)
- [HackTricks Cloud: AWS ElastiCache](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
