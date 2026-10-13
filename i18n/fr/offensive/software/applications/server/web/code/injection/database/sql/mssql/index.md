---
title: "Injection SQL Microsoft SQL Server (MSSQL) : techniques et payloads"
description: "Exploiter l'injection SQL contre Microsoft SQL Server : requêtes empilées, fuites par erreur de conversion, timing WAITFOR, xp_cmdshell, serveurs liés, et le catalogue sys."
keywords:
  - injection SQL MSSQL
  - injection T-SQL
  - xp_cmdshell
  - WAITFOR DELAY
  - erreur de conversion
  - serveurs liés
---

# MSSQL

Microsoft SQL Server utilise le dialecte T-SQL, et une injection réussie contre lui est souvent à fort impact car l'exécution de commandes, l'accès aux fichiers et le déplacement latéral vers d'autres serveurs sont tous atteignables depuis SQL.

Les commentaires sont `--` et `/* */`. Les chaînes se concatènent avec `+` (et `CONCAT()` depuis 2012), et `CHAR()`/`NCHAR()` construisent des chaînes sans guillemets. MSSQL autorise couramment les requêtes empilées : une seconde instruction séparée par `;` s'exécute généralement, ce qui rend `EXEC`, le DDL et les changements de configuration atteignables depuis une seule injection. Le catalogue réside dans le schéma `sys` (`sys.databases`, `sys.tables`, `sys.columns`, `sys.sql_logins`) et dans `information_schema`, et les métadonnées du serveur proviennent de `@@version`, `SERVERPROPERTY()`, `DB_NAME()` et `SYSTEM_USER`.

Deux détails de dialecte façonnent les payloads. Il n'y a pas de `LIMIT` ; la limitation de lignes se fait avec `TOP n` ou `OFFSET ... FETCH` (2012+), donc les payloads aveugles et union utilisent `TOP 1`. Et l'agrégation en une seule cellule utilise `STRING_AGG()` (2017+) ou l'astuce `FOR XML PATH('')` sur les versions plus anciennes, à la place du `GROUP_CONCAT` de MySQL.

L'impact dépend du rôle serveur du login. Un membre de `sysadmin` peut activer et exécuter `xp_cmdshell`, lire et écrire des fichiers, et pivoter via des serveurs liés, donc `IS_SRVROLEMEMBER('sysadmin')` est vérifié tôt.

## Techniques

- **[Énumération](enumeration.md)** : version, contexte courant et rôle serveur.
- **[Contournement d'authentification](authentication-bypass.md)** : subvertir un login construit à partir des champs d'identifiants.
- **[Basée sur union](union-based.md)** : ajouter un `UNION SELECT` avec des types correspondants.
- **[Basée sur les erreurs](error-based.md)** : divulguer des valeurs via des erreurs de conversion.
- **[Aveugle](blind.md)** : inférer les données à partir des différences de réponse booléennes.
- **[Temporelle](time-based.md)** : inférer les données avec `WAITFOR DELAY`.
- **[Requêtes empilées](stacked-query.md)** : exécuter des instructions supplémentaires après un `;`.
- **[Privilèges](privileges.md)** : lire les rôles serveur et base de données du login.
- **[Identifiants de base de données](database-credentials.md)** : dumper les hachages de mots de passe des logins.
- **[Manipulation de fichiers](file-manipulation.md)** : lire et écrire des fichiers avec OPENROWSET et bcp.
- **[Hors bande](out-of-band.md)** : exfiltrer et capturer des hachages via UNC avec `xp_dirtree`.
- **[Exécution de commandes](command-execution.md)** : exécuter des commandes OS avec `xp_cmdshell` ou l'automatisation OLE.
- **[Liens de confiance](trusted-links.md)** : pivoter vers les serveurs liés avec `openquery` et RPC.

## Outils

- **sqlmap** : détection et exploitation automatisées de l'injection SQL Server sur toutes les techniques.
- **ghauri** : alternative rapide avec forte évasion de WAF.
- **sqlcmd** : client officiel pour valider directement les payloads T-SQL.

## Références

- Microsoft SQL Server Documentation : system catalog views, functions, configuration
- OWASP Testing Guide : Testing for SQL Injection
