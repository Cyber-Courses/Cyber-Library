---
title: "SQL Database"
order: 1
description: "Attaquer Azure SQL Database : authentification Entra et SQL, portée du pare-feu, utilisateurs contenus et identité managée du serveur."
keywords:
  - Azure SQL
  - SQL Database
  - utilisateurs contenus
  - pare-feu
  - identité managée
---

# SQL Database

Azure SQL Database est un SQL Server managé accessible sur le port TCP 1433, protégé par un pare-feu au niveau du serveur et authentifié soit par des connexions SQL, soit par des jetons Microsoft Entra. Les questions offensives sont de savoir si le pare-feu vous laisse l'atteindre, quel identifiant ou jeton vous authentifie, et si le serveur logique porte une **identité managée** que vous pouvez piloter depuis l'intérieur de la base de données.

## Pages

- **[Accès](access.md)** : atteindre la base de données via l'authentification Entra, les connexions SQL, les utilisateurs contenus ou l'identité managée du serveur.

## Références

- [HackTricks Cloud : Azure SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : authentification Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-aad-overview)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
