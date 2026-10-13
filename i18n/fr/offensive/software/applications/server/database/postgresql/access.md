---
title: "Accès PostgreSQL : s'authentifier au cluster"
description: "Obtenir une session PostgreSQL authentifiée via des identifiants par défaut et faibles, l'authentification trust qui ne nécessite aucun mot de passe, et les rôles qui comptent, comme point de départ vers l'exécution de commandes et l'accès aux fichiers."
keywords:
  - accès PostgreSQL
  - authentification trust
  - pg_hba.conf
  - superutilisateur postgres
  - psql
---

# Accès

L'objectif est toute session authentifiée, idéalement avec un rôle **superutilisateur**. Le rôle d'administration par défaut est `postgres`, et des entrées `pg_hba.conf` mal configurées rendent fréquemment l'accès trivial.

## Obtenir une session

```bash
# Default/weak credentials for the postgres superuser
psql "host=<target> user=postgres password=postgres dbname=postgres"

# Spray credentials
hydra -L users.txt -P passwords.txt <target> postgres
```

## Authentification trust

`pg_hba.conf` peut associer un hôte ou un réseau à la méthode **`trust`**, qui accepte n'importe quel nom d'utilisateur **sans mot de passe**. Lorsque le service est exposé et qu'une ligne `trust` couvre votre source, vous vous connectez avec n'importe quel rôle, y compris `postgres`, gratuitement :

```bash
psql "host=<target> user=postgres dbname=postgres"   # no password prompt under trust
```

## Notes d'exploitation

- Vérifiez immédiatement les privilèges de votre rôle : `SELECT current_user, usesuper FROM pg_user WHERE usename = current_user;`. Un superutilisateur passe directement à l'[exécution de commandes](command-execution.md).
- Un non-superutilisateur reste utile : certains rôles (`pg_read_server_files`, `pg_execute_server_program`) accordent l'accès aux [fichiers](file-access.md) et aux programmes sans superutilisateur complet, et plusieurs chemins font passer un rôle ordinaire à superutilisateur.
- `trust` sur un cluster exposé à Internet constitue un point d'ancrage superutilisateur non authentifié, alors testez-le toujours avant de faire du spraying.

## Outils

- **psql** : le client natif pour l'accès interactif et les tests.
- **hydra / metasploit `postgres_login`** : spraying d'identifiants.

## Références

- [PostgreSQL: pg_hba.conf authentication methods](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
- [HackTricks: pentesting PostgreSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
