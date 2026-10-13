---
order: 3
title: "Injection SQL par moteur de base de données"
description: "L'injection SQL organisée par moteur de base de données, parce que la syntaxe exploitable, les fonctions, le catalogue et les primitives de fichier et de commande diffèrent nettement entre MySQL, PostgreSQL, SQL Server, Oracle, SQLite et Db2."
keywords:
  - injection SQL
  - dialectes de moteurs de base de données
  - techniques SQLi
  - information_schema
  - exploitation de base de données
---

# SQL

L'injection SQL survient lorsqu'une entrée non fiable est concaténée dans une requête SQL au lieu d'être liée comme paramètre, ce qui permet à un attaquant de changer le sens de la requête. Les techniques de haut niveau sont partagées, lectures basées sur union, fuites basées sur les erreurs, inférence aveugle booléenne et temporelle, exfiltration hors bande, et accès aux fichiers ou aux commandes, mais le payload exact qui implémente chacune est spécifique au moteur.

C'est pourquoi ce domaine est organisé par moteur de base de données plutôt que par technique seule. La syntaxe des commentaires, la concaténation de chaînes, le catalogue système, les fonctions de temporisation et les primitives de lecture de fichiers ou d'exécution de commandes diffèrent tous entre moteurs. Un payload qui fonctionne tel quel sur MySQL échoue sur PostgreSQL ou Oracle, donc savoir quel moteur se trouve derrière l'application décide de la syntaxe applicable. Chaque page de moteur couvre son fingerprinting puis parcourt les techniques dans son propre dialecte.

## Moteurs

- **[MySQL](mysql/index.md)** (et le compatible MariaDB) : `information_schema`, fonctions d'erreur XPath, `SLEEP`, et lecture et écriture conditionnées par `FILE`.
- **PostgreSQL** : dollar-quoting, fuites d'erreur par `CAST`, `pg_sleep`, et `COPY ... PROGRAM` pour l'exécution de commandes.
- **Microsoft SQL Server** : requêtes empilées, fuites par erreur de conversion, `WAITFOR DELAY`, et `xp_cmdshell`.
- **Oracle** : `FROM DUAL` obligatoire, packages `UTL_*`, et canaux hors bande conditionnés par ACL.
- **SQLite** : énumération `sqlite_master`, pas d'`information_schema`, et requêtes empilées dépendantes de l'API.
- **IBM Db2** : `SYSIBM.SYSDUMMY1`, registres spéciaux, et vues de catalogue `SYSCAT`.

## Sous-rubriques

- **[IBM Db2](ibm-db2/index.md)** : exploiter l'injection SQL contre IBM Db2 (LUW) : la table à ligne unique SYSIBM.SYSDUMMY1, le catalogue SYSCAT, les registres spéciaux, l'extraction LISTAGG, et l'absence de...
- **[MSSQL](mssql/index.md)** : exploiter l'injection SQL contre Microsoft SQL Server : requêtes empilées, fuites par erreur de conversion, timing WAITFOR, xp_cmdshell, serveurs liés, et le catalogue sys...
- **[Oracle](oracle/index.md)** : exploiter l'injection SQL contre Oracle Database : le FROM DUAL obligatoire, les packages UTL conditionnés par ACL, le timing DBMS_PIPE, les blocs PL/SQL, et les vues de catalogue ALL_.
- **[PostgreSQL](postgresql/index.md)** : exploiter l'injection SQL contre PostgreSQL : dollar-quoting, casts de chaînes pour les fuites d'erreur, timing pg_sleep, requêtes empilées, pg_read_file, et COPY ...
- **[SQLite](sqlite/index.md)** : exploiter l'injection SQL contre SQLite : énumération sqlite_master, typage souple, vraies primitives d'erreur, timing randomblob, écritures de fichiers ATTACH, et load_exte...

## Outils

- **sqlmap** : détection et exploitation automatisées sur tous les grands moteurs, avec des payloads par moteur.
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **Burp Suite** : le Scanner signale les points d'injection et le Repeater affine les payloads à la main.

## Références

- OWASP Testing Guide : Testing for SQL Injection
- PortSwigger Web Security Academy : SQL injection
