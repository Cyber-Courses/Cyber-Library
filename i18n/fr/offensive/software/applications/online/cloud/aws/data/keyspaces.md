---
title: "Keyspaces : atteindre des tables compatibles Cassandra"
order: 13
description: "Accéder à des tables Keyspaces (compatibles Cassandra) via des identifiants exposés et des autorisations trop larges."
keywords:
  - Keyspaces
  - Cassandra
  - CQL
  - table
  - identifiants
---

# Keyspaces

Keyspaces est la base de données compatible Cassandra d'AWS. On l'atteint soit avec des identifiants propres au service (un nom d'utilisateur et un mot de passe générés liés à un utilisateur IAM), soit avec la signature SigV4, et un principal détenant `cassandra:Select` lit toute table que la stratégie autorise. Les autorisations trop larges et les identifiants propres au service récupérables sont la voie d'entrée.

## Se connecter et lire

```bash
aws keyspaces list-keyspaces ; aws keyspaces list-tables --keyspace-name <ks>

# cqlsh with service-specific credentials over TLS (port 9142)
cqlsh cassandra.<region>.amazonaws.com 9142 --ssl \
  -u <svc-user> -p <svc-pass>
> SELECT * FROM ks.table LIMIT 100;
```

## Notes d'exploitation

- Les identifiants propres au service sont générés par utilisateur IAM et, une fois récupérés, donnent un accès CQL direct indépendant de toute vérification IAM ultérieure.
- Un plugin SigV4 pour le pilote Cassandra permet à un principal détenteur de se connecter sans aucun mot de passe stocké.
- Un `cassandra:Select` large au niveau du service lit toutes les tables des keyspaces du compte.

## Outils

- **AWS CLI** (`keyspaces list-keyspaces`, `list-tables`).
- **cqlsh** avec le plugin d'authentification SigV4 : accès CQL natif.

## Références

- [AWS: Keyspaces access and credentials](https://docs.aws.amazon.com/keyspaces/latest/devguide/security_iam_service-with-iam.html)
- [HackTricks Cloud: AWS Keyspaces](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [CloudFox: enumerating reachable cloud data stores](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
