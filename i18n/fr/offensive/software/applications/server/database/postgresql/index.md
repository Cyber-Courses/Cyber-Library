---
order: 2
title: "PostgreSQL"
description: "La surface offensive de PostgreSQL atteinte en tant que service : s'authentifier, exécuter des commandes système via COPY FROM PROGRAM et les langages procéduraux non fiables, lire et écrire des fichiers de l'hôte, et escalader vers le superutilisateur."
keywords:
  - PostgreSQL
  - COPY FROM PROGRAM
  - plpython
  - lo_export
  - superuser
---

# PostgreSQL

PostgreSQL (port 5432) est une cible fréquente car, en tant que **superutilisateur**, il exécute des commandes système et lit ou écrit des fichiers arbitraires sur l'hôte, le tout via des fonctionnalités documentées. Même en tant que non-superutilisateur, il existe des primitives de fichiers et de fonctions qui valent la peine d'être exploitées, et plusieurs chemins bien connus font passer un rôle ordinaire au statut de superutilisateur.

## Points d'entrée à viser

- **[Accès](access.md)** : identifiants par défaut et faibles, authentification `trust`, et obtention d'une connexion.
- **[Exécution de commandes](command-execution.md)** : `COPY ... FROM PROGRAM`, langages non fiables (`plpythonu`, `plperlu`), et extensions C.
- **[Accès aux fichiers](file-access.md)** : `COPY`, large objects et `pg_read_file` pour lire et écrire des fichiers de l'hôte.

## Références

- [HackTricks: pentesting PostgreSQL (5432)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
- [PostgreSQL: COPY](https://www.postgresql.org/docs/current/sql-copy.html)
