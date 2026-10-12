---
title: "Base de données"
order: 2
description: "Périmètre offensif des serveurs de base de données atteints en tant que service : s'authentifier, énumérer et élever ses privilèges à l'intérieur du moteur, exécuter des commandes du système d'exploitation depuis celui-ci, lire et écrire des fichiers de l'hôte, et pivoter via les liens de confiance, avec Microsoft SQL Server comme fer de lance intégré à AD."
keywords:
  - database
  - MSSQL
  - PostgreSQL
  - MySQL
  - Oracle
---

# Base de données

Un serveur de base de données est atteint en tant que service réseau, et une fois que vous pouvez vous authentifier, il est bien plus qu'un magasin de données : les principaux moteurs peuvent **exécuter des commandes du système d'exploitation**, **lire et écrire des fichiers sur l'hôte**, **usurper l'identité d'autres principaux de base de données** et **pivoter via les liens de confiance** vers d'autres serveurs. Dans un parc Windows, le service de base de données s'exécute également sous un compte privilégié et parle NTLM, ce qui en fait un point d'ancrage et une cible de déplacement latéral de premier plan, et pas seulement une source de données.

Ce domaine organise les attaques par moteur, car le langage de requête, le modèle de privilèges et les primitives d'exécution de commandes diffèrent fortement de l'un à l'autre.

## Vers quoi se tourner

- **Accès et énumération** : obtenir un login (par défaut, faible ou capturé), puis cartographier les bases de données, les rôles et les privilèges que vous détenez.
- **Exécution de commandes** : transformer un accès aux requêtes en code sur l'hôte.
- **Accès aux fichiers** : lire et écrire des fichiers de l'hôte via le moteur.
- **Élévation de privilèges à l'intérieur du moteur** : usurpation d'identité et chaînes de propriété pour atteindre des droits administratifs.
- **Déplacement latéral** : liens de confiance entre serveurs, et coercition de l'authentification du compte de service pour le relais.

## Moteurs

- **[MSSQL](mssql/index.md)** : Microsoft SQL Server, la cible la plus riche et la plus intégrée à AD : xp_cmdshell, serveurs liés, usurpation d'identité et coercition vers le relais NTLM.
- **[PostgreSQL](postgresql/index.md)** : exécution de commandes en tant que superuser via COPY FROM PROGRAM et les langages non fiables, plus lecture et écriture de fichiers.
- **[MySQL et MariaDB](mysql/index.md)** : écriture de fichier vers un webshell avec INTO OUTFILE et exécution de commandes OS via une fonction définie par l'utilisateur.
- **[Oracle Database](oracle/index.md)** : énumération TNS et SID, comptes par défaut, et exécution de commandes via le scheduler, Java et les tables externes.
- **[Redis](redis.md)** : accès non authentifié et les voies d'écriture de fichier et de chargement de module vers l'exécution de code.
- **[MongoDB](mongodb.md)** : exposition non authentifiée, énumération et JavaScript côté serveur lorsqu'il est activé.

## Références

- [HackTricks: pentesting MSSQL (1433)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [PayloadsAllTheThings: SQL injection and database notes](https://github.com/swisskyrepo/PayloadsAllTheThings)
