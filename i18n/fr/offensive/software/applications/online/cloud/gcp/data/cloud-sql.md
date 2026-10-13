---
title: "Cloud SQL : atteindre les bases de données via le proxy, l'IP publique et les utilisateurs intégrés"
order: 1
description: "Attaquer Cloud SQL : accès à la base de données via le SQL Auth Proxy, l'IP publique et les réseaux autorisés, et les utilisateurs de base de données intégrés."
keywords:
  - Cloud SQL
  - SQL Auth Proxy
  - réseaux autorisés
  - base de données
  - IP publique
  - MySQL
---

# Cloud SQL

Cloud SQL exécute des instances managées de MySQL, PostgreSQL et SQL Server. Atteindre les données est soit un problème réseau (une IP publique avec une plage de réseau autorisée large, ou le SQL Auth Proxy ouvert depuis un hôte que vous détenez), soit un problème d'identifiants (utilisateurs de base de données intégrés dont les mots de passe se trouvent dans les métadonnées, Secret Manager ou la configuration applicative, et l'authentification de base de données IAM où votre principal se voit accorder un rôle de base de données).

## Trouver et atteindre les instances

```bash
gcloud sql instances list
gcloud sql instances describe <inst> \
  --format='value(ipAddresses,settings.ipConfiguration.authorizedNetworks)'
# a public IP + 0.0.0.0/0 authorized network is directly reachable
```

## Se connecter

```bash
# built-in user over a reachable IP
mysql -h <public-ip> -u root -p

# or tunnel through the SQL Auth Proxy with your gcloud creds
cloud-sql-proxy <project>:<region>:<inst> &
psql "host=127.0.0.1 dbname=postgres user=postgres"

# IAM database authentication: mint a token as your principal
gcloud sql generate-login-token
```

## Notes d'exploitation

- `cloudsql.instances.connect` plus le proxy atteint une instance à IP privée sans mot de passe lorsque l'authentification de base de données IAM est activée pour votre principal.
- Les mots de passe des utilisateurs intégrés sont fréquemment réutilisés depuis [Secret Manager](../credentials/secret-manager.md) ou les [métadonnées](../credentials/instance-metadata/index.md) d'une instance ; récoltez-les en priorité.
- Un export de base de données (`gcloud sql export sql`) vers un bucket que vous contrôlez extrait toute la base hors ligne lorsque vous détenez `cloudsql.instances.export`.

## Outils

- **gcloud** (`sql instances list/describe`, `sql export sql`).
- **cloud-sql-proxy** : tunnel authentifié vers une instance.
- **clients natifs** (`mysql`, `psql`, `sqlcmd`).

## Références

- [HackTricks Cloud : GCP Cloud SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google Cloud : Cloud SQL Auth Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy)
