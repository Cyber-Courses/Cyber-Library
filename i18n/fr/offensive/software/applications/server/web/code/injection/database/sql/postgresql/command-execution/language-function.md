---
title: "Exécution de commandes PostgreSQL via des fonctions en langage non sécurisé"
description: "Définir une fonction en plpythonu, plperlu ou C depuis une injection PostgreSQL superutilisateur pour exécuter des commandes OS quand COPY FROM PROGRAM n'est pas disponible."
keywords:
  - plpythonu
  - plperlu
  - CREATE FUNCTION
  - libc system
  - RCE PostgreSQL
---

# Fonction en langage non sécurisé

Quand `COPY ... FROM PROGRAM` est bloqué mais que le rôle est superutilisateur, définir une fonction dans un langage non sécurisé donne l'exécution de commandes. Les langages non sécurisés (`plpythonu`, `plperlu` et C) s'exécutent sans bac à sable, leurs fonctions peuvent donc faire appel au système d'exploitation. Les créer nécessite un superutilisateur.

Avec un langage procédural non sécurisé installé, enveloppez un appel de sous-processus. L'extension moderne est PL/Python 3, enregistrée sous `plpython3u` (l'ancien `plpythonu` Python 2 a disparu sur les versions maintenues), confirmez donc d'abord le langage disponible dans `pg_language` :

```sql
' UNION SELECT string_agg(lanname,','),NULL,NULL FROM pg_language-- 
'; CREATE OR REPLACE FUNCTION exec(cmd text) RETURNS text AS $$ import subprocess; return subprocess.check_output(cmd, shell=True).decode() $$ LANGUAGE plpython3u-- 
' UNION SELECT exec('id'),NULL,NULL-- 
```

`plperlu` est l'équivalent là où Perl est le langage non sécurisé installé.

Une fonction en langage C est le repli quand aucun langage procédural non sécurisé n'est présent, mais ce n'est pas aussi simple que de se lier à la libc : PostgreSQL rejette tout objet partagé dépourvu de son bloc magique de module (`incompatible library ... missing magic block`), si bien que `CREATE FUNCTION ... AS '/lib/.../libc.so.6','system' LANGUAGE c` échoue sur les versions actuelles. Le vrai chemin C consiste à compiler une petite extension PostgreSQL (un `.so` construit avec `PG_MODULE_MAGIC` exportant une fonction d'exécution de commandes) et à la charger avec `CREATE FUNCTION`, ce qui explique pourquoi l'exécution de commandes repose en pratique sur `COPY ... FROM PROGRAM` ou un langage procédural non sécurisé plutôt que sur une simple liaison à la libc.

Chaque variante ici s'exécute en tant que compte du processus PostgreSQL et nécessite un superutilisateur, énumérez donc d'abord le rôle ; face à un non-superutilisateur, `COPY ... FROM PROGRAM` via le rôle `pg_execute_server_program` est le seul chemin de commande.

## Outils

- **sqlmap** : automatise l'exécution de commandes contre les rôles PostgreSQL privilégiés.
- **psql** : client officiel pour créer et appeler la fonction en langage non sécurisé.

## Références

- PostgreSQL Documentation: CREATE FUNCTION, procedural languages, C-language functions
- OWASP Testing Guide: Testing for SQL Injection
