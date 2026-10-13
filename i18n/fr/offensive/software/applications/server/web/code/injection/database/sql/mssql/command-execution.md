---
order: 2
title: "Exécution de commandes via injection MSSQL"
description: "Exécuter des commandes OS depuis une injection SQL Server privilégiée avec xp_cmdshell, y compris la séquence correcte de réactivation sp_configure, ainsi que l'automatisation OLE et les scripts externes."
keywords:
  - xp_cmdshell
  - sp_configure
  - automatisation OLE
  - sp_OACreate
  - sp_execute_external_script
---

# Exécution de commandes

SQL Server peut exécuter des commandes OS directement via des procédures étendues, ce qui fait d'une injection de niveau `sysadmin` une voie rapide vers l'exécution de code en tant que compte de service SQL Server.

La voie principale est `xp_cmdshell`. Elle est désactivée par défaut à partir de SQL Server 2005, elle est donc réactivée via `sp_configure` (qui nécessite `sysadmin`) puis appelée. Ce sont des instructions séparées, donc la chaîne nécessite des requêtes empilées :

```sql
'; EXEC sp_configure 'show advanced options',1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;-- 
'; EXEC xp_cmdshell 'whoami'-- 
```

Vérifiez son état actuel dans `sys.configurations` par nom plutôt que par un id numérique :

```sql
' UNION SELECT CAST(value_in_use AS varchar),name,NULL FROM sys.configurations WHERE name='xp_cmdshell'-- 
```

Quand `xp_cmdshell` est bloqué ou retiré, les procédures d'automatisation OLE exécutent une commande via un objet COM (également `sysadmin`, et l'option `Ole Automation Procedures` doit être activée) :

```sql
'; DECLARE @o int; EXEC sp_OACreate 'WScript.Shell',@o OUT; EXEC sp_OAMethod @o,'Run',NULL,'cmd /c whoami > C:\windows\temp\o.txt'-- 
```

Une troisième voie, là où Machine Learning Services est installé, est `sp_execute_external_script` exécutant Python ou R, dont le processus peut lancer des commandes. Toutes nécessitent `sysadmin` (ou un proxy soigneusement configuré), donc énumérez d'abord le rôle ; sans lui, poursuivez plutôt les voies fichier, identifiants ou serveur lié. Les commandes s'exécutent en tant que compte de service SQL Server, donc ses privilèges déterminent le point d'ancrage.

## Outils

- **sqlmap** : automatise la réactivation de `xp_cmdshell` et l'exécution de commandes avec `--os-shell`.
- **Metasploit Framework** : module `mssql_payload` pour l'exécution de commandes en tant que compte de service.
- **PowerUpSQL** : `Invoke-SQLOSCmd` pour l'exécution de commandes via xp_cmdshell et automatisation OLE.

## Références

- Microsoft SQL Server Documentation : xp_cmdshell, sp_configure, OLE Automation procedures
- OWASP Testing Guide : Testing for SQL Injection
